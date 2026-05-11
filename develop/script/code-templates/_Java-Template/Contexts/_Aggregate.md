<%*
const { className, columns, packageConfig } = tp.user.data;

const subdir = packageConfig.subdirectory ? `.${packageConfig.subdirectory}` : "";
const pkgInfra = `com.example.api.infrastructure${subdir}`;
const pkgContext = `com.example.api.contexts${subdir}`;

// 必須パラメータ = NULL:NO かつ PK・AI でないカラム
const requiredColumns = columns.filter(c =>
    !c.nullable &&
    !c.remark.includes("PK") &&
    !c.remark.includes("AI")
);

// created_at / updated_at カラムを自動検出
const createdCol     = columns.find(c => c.column === "created_at" && c.javaType === "LocalDateTime");
const updatedCol     = columns.find(c => c.column === "updated_at" && c.javaType === "LocalDateTime");
const hasDatetime    = createdCol || updatedCol;
const datetimeImport = hasDatetime ? `\nimport java.time.LocalDateTime;` : "";

// factoryNew のシグネチャ
const factoryNewParams = requiredColumns
    .map(c => `${c.javaType} ${c.field}`)
    .join(", ");

// factoryNew 内のEntityへのセット処理
const factoryNewBody = requiredColumns
    .map(c => `        entity.${c.field} = ${c.field};`)
    .join("\n");

const createdSet = createdCol ? `        entity.${createdCol.field} = LocalDateTime.now();` : "";
const updatedSet = updatedCol ? `        entity.${updatedCol.field} = LocalDateTime.now();` : "";

// factoryNew（必須パラメータが0件の場合は生成しない）
const factoryNew = requiredColumns.length > 0 ? `
    public static ${className}Aggregate factoryNew(${factoryNewParams}) {
        ${className} entity = new ${className}();
${factoryNewBody}
${createdSet}
${updatedSet}
        return new ${className}Aggregate(entity);
    }` : "";

// factoryUpdate 内の全フィールドコピー処理
const updatedSetForUpdate = updatedCol
    ? `        entity.${updatedCol.field} = LocalDateTime.now();`
    : "";

// factoryUpdate
const factoryUpdate = `
    public static ${className}Aggregate factoryUpdate(
            ${className}Aggregate aggregate /* TODO: 更新パラメータを追加 */) {
        ${className} current = aggregate.getEntity();
        ${className} entity  = new ${className}();
        BeanUtils.copyProperties(current, entity);
        // TODO: 更新項目を上書き
        //       例: entity.fieldName = newValue;
${updatedSetForUpdate}
        return new ${className}Aggregate(entity);
    }`;

tR += `package ${pkgContext}.domain.aggregate;

import ${pkgInfra}.entity.${className};${datetimeImport}
import org.springframework.beans.BeanUtils;

public class ${className}Aggregate {
    private final ${className} entity;

    public ${className}Aggregate(${className} entity) {
        this.entity = entity;
    }

    public ${className} getEntity() {
        return this.entity;
    }

    public static ${className}Aggregate factory(${className} entity) {
        return new ${className}Aggregate(entity);
    }
${factoryNew}
${factoryUpdate}
}`;
%>
