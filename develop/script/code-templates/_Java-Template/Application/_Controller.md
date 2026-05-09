<%*
const { apis, allTables, apiName, apiNameJp, url } = tp.user.data;

const toClassName = (tableName) =>
    tableName.split("_").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

const toPascalCase = (str) =>
    str.split("-").map(s => s.charAt(0).toUpperCase() + s.slice(1)).join("");

const classNames = allTables.map(toClassName);

// インポート
const entityImports = classNames.length > 0
    ? classNames.map(c => `import com.example.api.infrastructure.entity.${c};`).join("\n")
    : "";

const repositoryImports = classNames.length > 0
    ? classNames.map(c => `import com.example.api.infrastructure.repository.${c}Repository;`).join("\n")
    : "";

// @Autowiredフィールド
const autowiredFields = classNames.length > 0
    ? classNames.map(c => {
        const field = c.charAt(0).toLowerCase() + c.slice(1);
        return `    @Autowired\n    private ${c}Repository ${field}Repository;`;
    }).join("\n\n")
    : "";

// メソッド
const methods = apis.map(a => {
    const method     = a.http.toLowerCase();
    const annotation = `@${toPascalCase(method)}Mapping(API_URL)`;
    return `    /**
     * ${method}-${a.apiNameJp}
     */
    @Operation(summary = "${method}-${a.apiNameJp}")
    ${annotation}
    public void ${method}() throws Exception {
        // TODO: implement
    }`;
}).join("\n\n");

tR += `package com.example.api.application.controller;

${entityImports}
${repositoryImports}
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import org.springframework.beans.factory.annotation.Autowired;
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
