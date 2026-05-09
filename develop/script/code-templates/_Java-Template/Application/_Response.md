<%*
const { className, apiNameJp, responseFields, responseInnerClasses } = tp.user.data;

tR += `package com.example.api.application.resource.response;

/**
 * ${apiNameJp} - Response
 */
public class ${className}Response {
${responseFields}${responseInnerClasses ? "\n\n" + responseInnerClasses : ""}
}
`;
%>
