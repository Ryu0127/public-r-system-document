<%* 
const currentFolder = tp.file.folder(true); // 現在のノートのフォルダパスを取得
// クラス名をダイアログで入力 
const className = await tp.system.prompt("クラス名を入力"); 
const packageName = await tp.system.prompt("パッケージ名を入力", "com.example"); 
// Javaファイルの中身を生成 
const javaContent = `
package ${packageName}; 

public class ${className} { 
	public ${className}() { } 
	public static void main(String[] args) { } 
} `; 

// 現在のディレクトリ配下に作成 
const filePath = `${currentFolder}/${className}.java`; 
await app.vault.create(filePath, javaContent); 
tR += `✅ ${filePath} を作成しました！`;
%>