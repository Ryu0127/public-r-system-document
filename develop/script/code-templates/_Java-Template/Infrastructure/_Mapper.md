<%*
const { tableNameJp, className } = tp.user.data;

tR += `package com.example.api.infrastructure.mapper;

import com.example.api.infrastructure.entity.${className};  
import org.apache.ibatis.annotations.Mapper;  
import org.apache.ibatis.annotations.Param;  
  
import java.util.List;  
  
/**  
 * ${tableNameJp}
 *  
 */
@Mapper  
public interface ${className}Mapper {  
    /**  
     * レコードの取得（有効データの全件抽出）  
     * @return  
     */  
    List<${className}> all();  
  
    /**  
     * レコードの取得（主キー抽出）  
     * @return  
     */  
    ${className} find(@Param("id") Integer id);  
  
    /**  
     * レコードの取得（主キー複数抽出）  
     * @return  
     */  
    List<${className}> finds(@Param("ids") List<Integer> ids);  
  
    /**  
     * 登録（主キー生成）  
     */  
    void insert(${className} entity);  
  
    /**  
     * 更新（主キー対象）  
     */  
    void update(${className} entity);  
  
    /**  
     * 物理削除（主キー対象）  
     */  
    void delete(@Param("id") Integer id);  
}`;
%>