<%*
const targetPath = "PLACEHOLDER_PATH";
const templater = app.plugins.plugins["templater-obsidian"].templater;
const templateFile = app.vault.getAbstractFileByPath("develop/script/Create-Java-Infrastructure.md");
const targetFile = app.vault.getAbstractFileByPath(targetPath);
if (!templateFile) { new Notice("ERROR: Create-Java-Infrastructure.md が見つかりません"); return; }
if (!targetFile)   { new Notice("ERROR: 対象ファイルが見つかりません: " + targetPath); return; }
await templater.write_template_to_file(templateFile, targetFile);
%>
