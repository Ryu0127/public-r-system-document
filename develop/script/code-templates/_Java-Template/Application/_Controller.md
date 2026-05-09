<%*
const { apis, allTables, apiName, apiNameJp, url } = tp.user.data;

const toClassName = (tableName) =>
    tableName.split("_").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

const toPascalCase = (str) =>
    str.charAt(0).toUpperCase() + str.slice(1);

const classNames = allTables.map(toClassName);

// インポート：Entity / Repository
const entityImports = classNames
    .map(c => `import com.example.api.infrastructure.entity.${c};`)
    .join("\n");

const repositoryImports = classNames
    .map(c => `import com.example.api.infrastructure.repository.${c}Repository;`)
    .join("\n");

// インポート：Request / Response（メソッドごと）
const resourceImports = apis.map(a =>
    `import com.example.api.application.resource.request.${a.className}Request;\n` +
    `import com.example.api.application.resource.response.${a.className}Response;`
).join("\n");

// @Autowiredフィールド
const autowiredFields = classNames.map(c => {
    const field = c.charAt(0).toLowerCase() + c.slice(1);
    return `    @Autowired\n    private ${c}Repository ${field}Repository;`;
}).join("\n\n");

// メソッド（Request/Responseを引数・戻り値に使用）
const methods = apis.map(a => {
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
        // TODO: implement
        return new ${a.className}Response();
    }`;
}).join("\n\n");

tR += `package com.example.api.application.controller;

${entityImports}
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

${methods}
}
`;
%>
