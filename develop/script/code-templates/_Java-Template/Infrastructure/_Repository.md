<%*
const { tableNameJp, className } = tp.user.data;

tR += `package com.example.api.infrastructure.repository;

import com.example.api.contexts.domain.aggregate.${className}Aggregate;  
import com.example.api.contexts.domain.collection.aggregate.${className}AggregateList;
import com.example.api.infrastructure.entity.${className};  
import com.example.api.infrastructure.mapper.${className}Mapper;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Repository;  
  
import java.util.List;

/**  
 * ${tableNameJp}
 *  
 */
@Repository
public class ${className}Repository {
	@Autowired  
	private ${className}Mapper mapper;
  
	/**  
	 * レコードの取得（有効データの全件抽出）  
	 * @return  
	 */  
	public ${className}AggregateList all() {  
	    return ${className}AggregateList.factory(mapper.all());  
	}  
  
	/**  
	 * レコードの取得（主キー抽出）  
	 * @return  
	 */  
	public ${className}Aggregate find(Integer id) {  
	    return ${className}Aggregate.factory(mapper.find(id));  
	}  
  
	/**  
	 * レコードの取得（主キー複数抽出）  
	 * @return  
	 */  
	public ${className}AggregateList finds(List<Integer> ids) {  
	    return ${className}AggregateList.factory(mapper.finds(ids));  
	}   
	  
	/**  
	 * 登録（主キー生成）  
	 */  
	public void insertGeneratedKeys(${className}Aggregate aggregate) {  
	    mapper.insertGeneratedKeys(aggregate.getEntity());  
	}  
	  
	/**  
	 * 更新 
	 */  
	public void update(${className}Aggregate aggregate) {  
	    mapper.update(aggregate.getEntity());  
	}  
	  
	/**  
	 * 物理削除 
	 */  
	public void delete(${className}Aggregate aggregate) {  
	    mapper.delete(aggregate.getEntity().id);  
	}
}`;
%>