<%*
const { className, apiNameJp, requestFields, requestInnerClasses } = tp.user.data;

tR += `package com.example.api.application.resource.request;

/**
 * ${apiNameJp} - Request
 */
public class ${className}Request {
${requestFields}${requestInnerClasses ? "\n\n" + requestInnerClasses : ""}
}
`;
%>
