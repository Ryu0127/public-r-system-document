<%*
const { tableNameJp, className, columns, packageConfig } = tp.user.data;

const pkg = packageConfig.subdirectory 
    ? `com.example.api.infrastructure.entity.${packageConfig.subdirectory}`
    : `com.example.api.infrastructure.entity`;

tR += `package ${pkg};

import lombok.Data;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;

/**  
 * ${tableNameJp}  
 *  
 */
@Data
public class ${className} {
${columns.map(column => `    public ${column.javaType} ${column.field}; // ${column.remark}`).join("\n")}
}`;
%>