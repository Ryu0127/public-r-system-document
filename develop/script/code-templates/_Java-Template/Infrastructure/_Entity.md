<%*
const { tableNameJp, className, columns } = tp.user.data;

tR += `package com.example.api.infrastructure.entity;

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