<%*
// =====================
// ユーティリティ関数
// =====================
const getTimestamp = () => {
    const now = new Date();
    return now.getFullYear().toString()
        + (now.getMonth() + 1).toString().padStart(2, "0")
        + now.getDate().toString().padStart(2, "0")
        + "_"
        + now.getHours().toString().padStart(2, "0")
        + now.getMinutes().toString().padStart(2, "0");
};

const toField = (columnName) =>
    columnName.replace(/_([a-z])/g, (_, c) => c.toUpperCase());

// =====================
// パッケージ構成関数
// =====================
const getPackageConfig = (fm) => {
    const basePackage = "com.example.api";
    const subdir = fm.subdirectory && fm.subdirectory.trim() 
        ? "." + fm.subdirectory.toLowerCase() 
        : "";
    const fullPackage = basePackage + subdir;
    const packagePath = fullPackage.replace(/\./g, "/");
    return { basePackage, subdir, fullPackage, packagePath };
};

// =====================
// MDパース関数
// =====================
const getFrontmatter = () => {
    const cache = app.metadataCache.getFileCache(tp.file.find_tfile(tp.file.title));
    return cache?.frontmatter;
};

const parseColumns = async () => {
    const content = await app.vault.read(tp.file.find_tfile(tp.file.title));
    return content.split("\n")
        .filter(l => l.startsWith("|") && !l.includes("カラム名") && !l.match(/\|[-| ]+\|/))
        .map(line => {
            const cols = line.split("|").map(s => s.trim()).filter(Boolean);
            return {
                column:   cols[0],
                sqlType:  cols[1],
                javaType: cols[2],
                nullable: cols[3] === "YES",
                remark:   cols[4] ?? "",
                field:    toField(cols[0])
            };
        });
};

// =====================
// フォルダ作成関数
// =====================
const createOutputFolder = async (baseFolder, packageConfig) => {
    const folderPath = `${baseFolder}/../../../generate-code/${getTimestamp()}`;
    const basePkgPath = "com/example/api";
    const subdir = packageConfig.subdirectory ? `/${packageConfig.subdirectory}` : "";
    
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/infrastructure/entity${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/infrastructure/repository${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/infrastructure/mapper${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/resources/${basePkgPath}/infrastructure/mapper${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/contexts/domain/aggregate${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/contexts/domain/collection/aggregate${subdir}`);
    return { folderPath, ...packageConfig };
};

// =====================
// ファイル生成関数
// =====================
const generateFiles = async (outputFolder, fm, packageConfig) => {
    const entityContent    = await tp.file.include("[[_Entity]]");
    const repositoryContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Infrastructure/_Repository]]");
    const mapperContent    = await tp.file.include("[[_Mapper]]");
    const mapperXmlContent = await tp.file.include("[[_MapperXml]]");
    const aggregateContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Contexts/_Aggregate]]");
    const aggregateListContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Contexts/_AggregateList]]");

    const basePkgPath = "com/example/api";
    const subdir = packageConfig.subdirectory ? `/${packageConfig.subdirectory}` : "";
    
    await app.vault.create(`${outputFolder.folderPath}/src/main/java/${basePkgPath}/infrastructure/entity${subdir}/${fm.className}.java`, entityContent);
    await app.vault.create(`${outputFolder.folderPath}/src/main/java/${basePkgPath}/infrastructure/repository${subdir}/${fm.className}Repository.java`, repositoryContent);
    await app.vault.create(`${outputFolder.folderPath}/src/main/java/${basePkgPath}/infrastructure/mapper${subdir}/${fm.className}Mapper.java`, mapperContent);
    await app.vault.create(`${outputFolder.folderPath}/src/main/resources/${basePkgPath}/infrastructure/mapper${subdir}/${fm.className}Mapper.xml`, mapperXmlContent);
    await app.vault.create(`${outputFolder.folderPath}/src/main/java/${basePkgPath}/contexts/domain/aggregate${subdir}/${fm.className}Aggregate.java`, aggregateContent);
    await app.vault.create(`${outputFolder.folderPath}/src/main/java/${basePkgPath}/contexts/domain/collection/aggregate${subdir}/${fm.className}AggregateList.java`, aggregateListContent);

    return [
        `${fm.className}.java`,
        `${fm.className}Repository.java`,
        `${fm.className}Mapper.java`,
        `${fm.className}Mapper.xml`,
        `${fm.className}Aggregate.java`,
        `${fm.className}AggregateList.java`,
    ];
};

// =====================
// メイン処理
// =====================
const main = async () => {
    const fm = getFrontmatter();
    const columns = await parseColumns();
    const pk = columns.find(c => c.remark.includes("PK"));
    const packageConfig = getPackageConfig(fm);

    // テンプレートへのデータ共有
    tp.user.data = {
	    tableName: fm.tableName,
	    tableNameJp: fm.tableNameJp,
	    className: fm.className,
	    subdirectory: fm.subdirectory,
	    columns, pk,
	    packageConfig
	};

    const outputFolder = await createOutputFolder(tp.file.folder(true), packageConfig);
    const createdFiles = await generateFiles(outputFolder, fm, packageConfig);
};

new Notice("✅ ファイルの生成が完了しました！", 5000); // 5秒表示

await main();
%>