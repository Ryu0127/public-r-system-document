---
apiNameJp: 
apiName: 
url: 
HTTP: 
subdirectory:
---

### API-Schema
---

### API-RequestBody
---
```json
```

### API-Response
---
```json
```

### DB-SELECT
---

### DB-INSERT/UPDATE/DELETE
---

### 外部API-GET
---

### シーケンス図
---

```dataviewjs
const btn = this.container.createEl("button", { text: "コード自動生成" });
btn.style.cssText = "padding:20px 16px; background:#7c3aed; color:white; border:none; border-radius:6px; cursor:pointer;";

btn.onclick = async () => {
    btn.setText("⏳ 生成中...");
    btn.disabled = true;
    try {
        const templater = app.plugins.plugins["templater-obsidian"].templater;
        const templateFile = app.vault.getAbstractFileByPath("develop/script/Create-Java-Controller.md");
        if (!templateFile) {
            throw new Error("テンプレートファイルが見つかりません: develop/script/Create-Java-Controller.md");
        }
        const targetFile = app.vault.getAbstractFileByPath(dv.current().file.path);
        await templater.write_template_to_file(templateFile, targetFile);
        btn.setText("✅ 完了！");
    } catch(e) {
        btn.setText("❌ エラー");
        console.error("詳細エラー:", e.message);
    } finally {
        btn.disabled = false;
    }
};
```
※コードは「/develop/generate-code/{作成年月日_時刻}」内に作成されます
