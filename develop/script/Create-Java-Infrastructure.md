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
const createOutputFolder = async (baseFolder) => {
    const folderPath = `${baseFolder}/../generate-code/${getTimestamp()}`;
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/infrastructure/entity`);
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/infrastructure/repository`);
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/infrastructure/mapper`);
    await app.vault.createFolder(`${folderPath}/src/main/resources/com/example/api/infrastructure/mapper`);
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/contexts/domain/aggregate`);
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/contexts/domain/collection/aggregate`);
    return folderPath;
};

// =====================
// ファイル生成関数
// =====================
const generateFiles = async (outputFolder, fm) => {
    const entityContent    = await tp.file.include("[[_Entity]]");
    const repositoryContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Infrastructure/_Repository]]");
    const mapperContent    = await tp.file.include("[[_Mapper]]");
    const mapperXmlContent = await tp.file.include("[[_MapperXml]]");
    const aggregateContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Contexts/_Aggregate]]");
    const aggregateListContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Contexts/_AggregateList]]");

    await app.vault.create(`${outputFolder}/src/main/java/com/example/api/infrastructure/entity/${fm.className}.java`, entityContent);
    await app.vault.create(`${outputFolder}/src/main/java/com/example/api/infrastructure/repository/${fm.className}Repository.java`, repositoryContent);
    await app.vault.create(`${outputFolder}/src/main/java/com/example/api/infrastructure/mapper/${fm.className}Mapper.java`, mapperContent);
    await app.vault.create(`${outputFolder}/src/main/resources/com/example/api/infrastructure/mapper/${fm.className}Mapper.xml`, mapperXmlContent);
    await app.vault.create(`${outputFolder}/src/main/java/com/example/api/contexts/domain/aggregate/${fm.className}Aggregate.java`, aggregateContent);
    await app.vault.create(`${outputFolder}/src/main/java/com/example/api/contexts/domain/collection/aggregate/${fm.className}AggregateList.java`, aggregateListContent);

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

    // テンプレートへのデータ共有
    tp.user.data = {
	    tableName: fm.tableName,
	    tableNameJp: fm.tableNameJp,
	    className: fm.className,
	    columns, pk
	};

    const outputFolder = await createOutputFolder(tp.file.folder(true));
    const createdFiles = await generateFiles(outputFolder, fm);
};

new Notice("✅ ファイルの生成が完了しました！", 5000); // 5秒表示

await main();
%>