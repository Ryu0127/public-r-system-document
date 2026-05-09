<%*
const { packageName, className, tableName, columns, pk } = tp.user.data;

tR += `<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.api.infrastructure.mapper.${className}Mapper">
	<sql id="tableName">${tableName}</sql>
    <resultMap id="${className}ResultMap" type="com.example.api.infrastructure.entity.${className}">
${columns.map(c => `        <result property="${c.field}" column="${c.column}" />`).join("\n")}
    </resultMap>
    <select id="all" resultMap="${className}ResultMap">
	    /* レコードの取得（全件抽出） */
        SELECT *
        FROM <include refid="tableName" />
    </select>
    <select id="find" resultMap="${className}ResultMap">
	    /* レコードの取得（主キー抽出） */
        SELECT *
        FROM <include refid="tableName" />
        WHERE ID = #{id}
    </select>
	<select id="finds" resultMap="${className}ResultMap">  
		/* レコードの取得（主キー複数抽出） */  
	    SELECT *  
	    FROM <include refid="tableName" />  
	    WHERE ID IN  
	    <foreach collection="ids" item="id" open="(" close=")" index="i" separator=",">  
		    #{id}  
		</foreach>  
	</select>
	<insert id="insertGeneratedKeys" parameterType="com.example.api.infrastructure.entity.TblEvent" useGeneratedKeys="true" keyProperty="id">  
        /* 登録（主キー生成） */  
        INSERT INTO <include refid="tableName" />  
        (
            ${columns.map(c => c.column).join(",\n            ")}
        ) VALUES (
            ${columns.map(c => `#{${c.field}}`).join(",\n            ")}
        )
    </insert>
    <update id="update">
	    /* 更新（主キー対象） */
        UPDATE <include refid="tableName" /> SET
            ${columns.filter(c => !c.remark.includes("PK")).map(c =>
                `${c.column} = #{${c.field}}`
            ).join(",\n            ")}
        WHERE ID = #{id}
    </update>
    <delete id="delete">
	    /* 物理削除（主キー対象） */
        DELETE FROM <include refid="tableName" />
        WHERE ID = #{id}
    </delete>

</mapper>`;
%>