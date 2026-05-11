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

const toPascalCase = (str) =>
    str.charAt(0).toUpperCase() + str.slice(1);

const toClassName = (tableName) =>
    tableName.split("_").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

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
// JSON → Javaフィールド変換
// =====================
const inferJavaType = (key, value) => {
    if (value === null)                          return "String";
    if (typeof value === "boolean")              return "Boolean";
    if (typeof value === "number")               return Number.isInteger(value) ? "Integer" : "Double";
    if (typeof value === "string")               return "String";
    if (Array.isArray(value))                    return "List<Object>";
    if (typeof value === "object")               return toPascalCase(key);
    return "String";
};

const buildFields = (obj, indent = "    ") => {
    if (!obj || typeof obj !== "object") return "";
    return Object.entries(obj)
        .map(([key, val]) => `${indent}public ${inferJavaType(key, val)} ${key};`)
        .join("\n");
};

const buildInnerClasses = (obj, indent = "    ") => {
    if (!obj || typeof obj !== "object") return "";
    return Object.entries(obj)
        .filter(([_, val]) => val && typeof val === "object" && !Array.isArray(val))
        .map(([key, val]) => {
            const fields = buildFields(val, indent + "    ");
            return `${indent}public static class ${toPascalCase(key)} {\n${fields}\n${indent}}`;
        })
        .join("\n\n");
};

// =====================
// ディレクトリ内の全APIファイルを取得
// =====================
const getApiFiles = (dirPath) => {
    const folder = app.vault.getAbstractFileByPath(dirPath);
    if (!folder || !folder.children) return [];
    return folder.children.filter(f => f.extension === "md");
};

// =====================
// APIファイルのパース
// =====================
const parseApiFile = async (file) => {
    const cache   = app.metadataCache.getFileCache(file);
    const fm      = cache?.frontmatter ?? {};
    const content = await app.vault.read(file);

    const extractJson = (label) => {
        const regex = new RegExp(
            `### ${label}\\n---\\n\`\`\`json\\n([\\s\\S]*?)\`\`\``, "m"
        );
        const match = content.match(regex);
        if (!match || !match[1].trim()) return null;
        try { return JSON.parse(match[1].trim()); } catch { return null; }
    };

    const extractLinks = (label) => {
        const regex = new RegExp(`### ${label}\\n---\\n([\\s\\S]*?)(?=\\n###|$)`);
        const match = content.match(regex);
        if (!match) return [];
        return [...match[1].matchAll(/\[\[(.+?)（.+?）\]\]/g)].map(m => m[1]);
    };

    const http      = fm.HTTP ?? "";
    const httpPascal = http.charAt(0).toUpperCase() + http.slice(1).toLowerCase();
    const className = `${fm.apiName}${httpPascal}`;

    const requestJson  = extractJson("API-RequestBody");
    const responseJson = extractJson("API-Response");

    return {
        apiNameJp:            fm.apiNameJp ?? "",
        apiName:              fm.apiName   ?? "",
        url:                  fm.url       ?? "",
        http,
        httpPascal,
        className,
        selectTables:         extractLinks("DB-SELECT"),
        writeTables:          extractLinks("DB-INSERT\\/UPDATE\\/DELETE"),
        requestJson,
        responseJson,
        requestFields:        buildFields(requestJson),
        requestInnerClasses:  buildInnerClasses(requestJson),
        responseFields:       buildFields(responseJson),
        responseInnerClasses: buildInnerClasses(responseJson),
    };
};

// =====================
// フォルダ作成関数
// =====================
const createOutputFolder = async (baseFolder, packageConfig) => {
    const folderPath = `${baseFolder}/../../../../generate-code/${getTimestamp()}`;
    const basePkgPath = "com/example/api";
    const subdir = packageConfig.subdirectory ? `/${packageConfig.subdirectory}` : "";
    
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/application/controller${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/application/resource/request${subdir}`);
    await app.vault.createFolder(`${folderPath}/src/main/java/${basePkgPath}/application/resource/response${subdir}`);
    return { folderPath, ...packageConfig };
};

// =====================
// メイン処理
// =====================
const main = async () => {
    const currentDir = tp.file.folder(true);
    const files      = getApiFiles(currentDir);

    if (files.length === 0) {
        new Notice("ERROR: APIファイルが見つかりません");
        return;
    }

    const apis         = await Promise.all(files.map(parseApiFile));
    const allTables    = [...new Set([
        ...apis.flatMap(a => a.selectTables),
        ...apis.flatMap(a => a.writeTables),
    ])];
    
    // フロントマターからパッケージ構成を取得
    const firstFileCache = app.metadataCache.getFileCache(files[0]);
    const fm = firstFileCache?.frontmatter ?? {};
    const packageConfig = getPackageConfig(fm);
    
    const outputFolder = await createOutputFolder(currentDir, packageConfig);
    const basePkgPath = "com/example/api";
    const subdir = packageConfig.subdirectory ? `/${packageConfig.subdirectory}` : "";

    // --- Controller ---
    tp.user.data = {
        apis,
        allTables,
        apiName:   apis[0].apiName,
        apiNameJp: apis[0].apiNameJp,
        url:       apis[0].url,
        packageConfig,
    };
    const controllerContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Application/_Controller]]");
    await app.vault.create(
        `${outputFolder.folderPath}/src/main/java/${basePkgPath}/application/controller${subdir}/${apis[0].apiName}Controller.java`,
        controllerContent
    );

    // --- Request / Response（メソッドごと） ---
    for (const api of apis) {
        tp.user.data = {
            className:            api.className,
            apiNameJp:            api.apiNameJp,
            requestFields:        api.requestFields,
            requestInnerClasses:  api.requestInnerClasses,
            responseFields:       api.responseFields,
            responseInnerClasses: api.responseInnerClasses,
            packageConfig,
        };

        const requestContent  = await tp.file.include("[[develop/script/code-templates/_Java-Template/Application/_Request]]");
        const responseContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Application/_Response]]");

        await app.vault.create(
            `${outputFolder.folderPath}/src/main/java/${basePkgPath}/application/resource/request${subdir}/${api.className}Request.java`,
            requestContent
        );
        await app.vault.create(
            `${outputFolder.folderPath}/src/main/java/${basePkgPath}/application/resource/response${subdir}/${api.className}Response.java`,
            responseContent
        );
    }

    new Notice(`✅ Controller / Request / Response の生成が完了しました！`, 5000);
};

await main();
%>
