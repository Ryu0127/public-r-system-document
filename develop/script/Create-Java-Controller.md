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

const toClassName = (tableName) =>
    tableName.split("_").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

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
    const cache = app.metadataCache.getFileCache(file);
    const fm    = cache?.frontmatter ?? {};
    const content = await app.vault.read(file);

    const extractLinks = (label) => {
        const regex = new RegExp(`### ${label}\\n---\\n([\\s\\S]*?)(?=\\n###|$)`);
        const match = content.match(regex);
        if (!match) return [];
        return [...match[1].matchAll(/\[\[(.+?)（.+?）\]\]/g)].map(m => m[1]);
    };

    return {
        apiNameJp:    fm.apiNameJp ?? "",
        apiName:      fm.apiName   ?? "",
        url:          fm.url       ?? "",
        http:         fm.HTTP      ?? "",
        selectTables: extractLinks("DB-SELECT"),
        writeTables:  extractLinks("DB-INSERT\\/UPDATE\\/DELETE"),
    };
};

// =====================
// フォルダ作成関数
// =====================
const createOutputFolder = async (baseFolder) => {
    // develop/api/doc/{urlPath}/{apiNameJp}/ から4階層上 = develop/
    const folderPath = `${baseFolder}/../../../../generate-code/${getTimestamp()}`;
    await app.vault.createFolder(`${folderPath}/src/main/java/com/example/api/application/controller`);
    return folderPath;
};

// =====================
// メイン処理
// =====================
const main = async () => {
    const currentDir = tp.file.folder(true);
    const files = getApiFiles(currentDir);

    if (files.length === 0) {
        new Notice("ERROR: APIファイルが見つかりません");
        return;
    }

    const apis = await Promise.all(files.map(parseApiFile));

    // 使用テーブルを収集（重複排除）
    const allTables = [...new Set([
        ...apis.flatMap(a => a.selectTables),
        ...apis.flatMap(a => a.writeTables),
    ])];

    // テンプレートへのデータ共有
    tp.user.data = {
        apis,
        allTables,
        apiName:   apis[0].apiName,
        apiNameJp: apis[0].apiNameJp,
        url:       apis[0].url,
    };

    const outputFolder     = await createOutputFolder(currentDir);
    const controllerContent = await tp.file.include("[[develop/script/code-templates/_Java-Template/Application/_Controller]]");

    await app.vault.create(
        `${outputFolder}/src/main/java/com/example/api/application/controller/${apis[0].apiName}Controller.java`,
        controllerContent
    );

    new Notice(`✅ ${apis[0].apiName}Controller.java の生成が完了しました！`, 5000);
};

await main();
%>
