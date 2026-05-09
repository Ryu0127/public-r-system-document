<%*
const { tableNameJp, className, tableName, columns } = tp.user.data;

tR += `<?php

namespace App\\Infrastructure\\Models;

use Illuminate\\Database\\Eloquent\\Model;

/**
 * ${tableNameJp}
 *
${columns.map(c => ` * @property ${c.phpType} $${c.field} // ${c.remark}`).join("\n")}
 */
class ${className} extends Model
{
    protected $table = '${tableName}';

    protected $fillable = [
${columns.filter(c => !c.remark.includes("PK")).map(c => `        '${c.field}',`).join("\n")}
    ];
}`;
%>
