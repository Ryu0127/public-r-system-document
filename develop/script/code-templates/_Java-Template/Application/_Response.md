<%*
const { className, apiNameJp, responseFields, responseInnerClasses, responseJson, selectClassNames } = tp.user.data;

// Aggregateのimport
const aggregateImports = (selectClassNames || [])
    .map(c => `import com.example.api.contexts.domain.aggregate.${c}Aggregate;`)
    .join("\n");

// トップレベルのプリミティブフィールド
const primitiveEntries = responseJson
    ? Object.entries(responseJson).filter(([_, val]) =>
        val === null || typeof val !== "object" || Array.isArray(val))
    : [];

// 内部クラスエントリ
const innerEntries = responseJson
    ? Object.entries(responseJson).filter(([_, val]) =>
        val && typeof val === "object" && !Array.isArray(val))
    : [];

const hasInnerClass = innerEntries.length > 0;
const hasAggregate  = selectClassNames && selectClassNames.length > 0;

// Javaの型推論
const inferType = (val) =>
    val === null ? "String"
    : typeof val === "boolean" ? "Boolean"
    : typeof val === "number" ? (Number.isInteger(val) ? "Integer" : "Double")
    : "String";

// buildメソッドのプリミティブ引数
const primitiveParams = primitiveEntries
    .map(([key, val]) => `${inferType(val)} ${key}`)
    .join(", ");

// Aggregateの引数（内部クラスがある場合）
const aggregateParams = (hasInnerClass && hasAggregate)
    ? selectClassNames
        .map(c => `${c}Aggregate ${c.charAt(0).toLowerCase() + c.slice(1)}Aggregate`)
        .join(", ")
    : "";

// buildの全引数
const allBuildParams = [primitiveParams, aggregateParams].filter(Boolean).join(", ");

// buildメソッド内のフィールドセット
const primitiveSetLines = primitiveEntries
    .map(([key]) => `        response.${key} = ${key};`)
    .join("\n");

// buildメソッド内でbuildBodyを呼び出す
const innerSetLines = (hasInnerClass && hasAggregate)
    ? innerEntries.map(([key]) => {
        const innerClassName = key.charAt(0).toUpperCase() + key.slice(1);
        const aggregateArgs  = selectClassNames
            .map(c => `${c.charAt(0).toLowerCase() + c.slice(1)}Aggregate`)
            .join(", ");
        return `        response.${key} = build${innerClassName}(${aggregateArgs});`;
    }).join("\n")
    : "";

// buildBodyメソッド
const innerBuildMethods = (hasInnerClass && hasAggregate)
    ? innerEntries.map(([key, val]) => {
        const innerClassName  = key.charAt(0).toUpperCase() + key.slice(1);
        const aggregateParams = selectClassNames
            .map(c => `${c}Aggregate ${c.charAt(0).toLowerCase() + c.slice(1)}Aggregate`)
            .join(", ");
        const entityLines = selectClassNames
            .map(c => {
                const field = c.charAt(0).toLowerCase() + c.slice(1);
                return `        ${c} ${field} = ${field}Aggregate.getEntity();`;
            })
            .join("\n");
        const fieldSetLines = Object.keys(val)
            .map(k => `        ${key}.${k} = // TODO: ${k};`)
            .join("\n");

        return `    private static ${innerClassName} build${innerClassName}(${aggregateParams}) {
${entityLines}
        ${innerClassName} ${key} = new ${innerClassName}();
${fieldSetLines}
        return ${key};
    }`;
    }).join("\n\n")
    : "";

tR += `package com.example.api.application.resource.response;

${aggregateImports}

/**
 * ${apiNameJp} - Response
 */
public class ${className}Response {
${responseFields}${responseInnerClasses ? "\n\n" + responseInnerClasses : ""}

    public static ${className}Response build(${allBuildParams}) {
        ${className}Response response = new ${className}Response();
${primitiveSetLines}
${innerSetLines}
        return response;
    }
${innerBuildMethods ? "\n" + innerBuildMethods : ""}
}
`;
%>
