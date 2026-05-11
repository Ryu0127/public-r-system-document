<%*
const { className, apiNameJp, requestFields, requestInnerClasses, packageConfig } = tp.user.data;

const subdir = packageConfig.subdirectory ? `.${packageConfig.subdirectory}` : "";
const pkg = `com.example.api.application.resource.request${subdir}`;

tR += `package ${pkg};

/**
 * ${apiNameJp} - Request
 */
public class ${className}Request {
${requestFields}${requestInnerClasses ? "\n\n" + requestInnerClasses : ""}
}
`;
%>
