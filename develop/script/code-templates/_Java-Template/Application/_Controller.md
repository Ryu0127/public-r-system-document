<%*
const { apis, allTables, apiName, apiNameJp, url } = tp.user.data;

const toClassName = (tableName) =>
    tableName.split("_").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

const toPascalCase = (str) =>
    str.charAt(0).toUpperCase() + str.slice(1);

const classNames = allTables.map(toClassName);

// インポート：Entity
const entityImports = classNames
    .map(c => `import com.example.api.infrastructure.entity.${c};`)
    .join("\n");

// インポート：Aggregate / AggregateList
const aggregateImports = classNames
    .map(c => [
        `import com.example.api.contexts.domain.aggregate.${c}Aggregate;`,
        `import com.example.api.contexts.domain.collection.aggregate.${c}AggregateList;`
    ].join("\n"))
    .join("\n");

// インポート：Repository
const repositoryImports = classNames
    .map(c => `import com.example.api.infrastructure.repository.${c}Repository;`)
    .join("\n");

// インポート：Request / Response
const resourceImports = apis.map(a =>
    `import com.example.api.application.resource.request.${a.className}Request;\n` +
    `import com.example.api.application.resource.response.${a.className}Response;`
).join("\n");

// @Autowiredフィールド
const autowiredFields = classNames.map(c => {
    const field = c.charAt(0).toLowerCase() + c.slice(1);
    return `    @Autowired\n    private ${c}Repository ${field}Repository;`;
}).join("\n\n");

// publicメソッド（エントリーポイント）
const publicMethods = apis.map(a => {
    const method     = a.http.toLowerCase();
    const annotation = `@${toPascalCase(method)}Mapping(API_URL)`;
    const hasRequest = a.requestJson && Object.keys(a.requestJson).length > 0;
    const requestArg = hasRequest ? `@RequestBody ${a.className}Request request` : "";

    return `    /**
     * ${method}-${a.apiNameJp}
     */
    @Operation(summary = "${method}-${a.apiNameJp}")
    ${annotation}
    public ${a.className}Response ${method}(${requestArg}) throws Exception {
        // ① SELECT
        // TODO: 必要に応じてSELECT処理を追加

        // ② Aggregate作成・更新
        // TODO: Aggregateの作成・更新処理を追加

        // ③ 永続化
        // TODO: 永続化処理を追加

        // TODO: Response.build() の引数にプリミティブ値とAggregateを渡す
        return ${a.className}Response.build(/* TODO: statusCode, message, aggregate */);
    }`;
}).join("\n\n");

// privateメソッド（Aggregate生成・Response生成）
const privateMethods = apis.map(a => {
    const classNames2 = [...new Set([...a.selectTables, ...a.writeTables])].map(toClassName);
    const hasRequest  = a.requestJson && Object.keys(a.requestJson).length > 0;

    // Aggregate生成メソッド（INSERT/UPDATE系のみ）
    const aggregateMethods = a.writeTables.map(t => {
        const c     = toClassName(t);
        const field = c.charAt(0).toLowerCase() + c.slice(1);
        const requestParam = hasRequest ? `${a.className}Request request` : "";
        return `    private ${c}Aggregate create${c}Aggregate(${requestParam}) {
        // TODO: factoryNew の引数に request の必須フィールドを渡す
        //       created_at / updated_at は factoryNew 内で自動セットされる
        return ${c}Aggregate.factoryNew(/* TODO: request.field */);
    }`;
    }).join("\n\n");

    return [aggregateMethods].filter(Boolean).join("\n\n");
}).join("\n\n");

tR += `package com.example.api.application.controller;

${entityImports}
${aggregateImports}
${repositoryImports}
${resourceImports}
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * ${apiNameJp}
 * ${url}
 */
@Tag(name = "${apiNameJp}")
@RestController
public class ${apiName}Controller {
    private static final String API_URL = "${url}";

${autowiredFields}

    // =====================
    // public
    // =====================
${publicMethods}

    // =====================
    // private
    // =====================
${privateMethods}
}
`;
%>
