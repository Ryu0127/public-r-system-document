<%*
const { className } = tp.user.data;

tR += `package com.example.api.contexts.domain.aggregate;  

import com.example.api.infrastructure.entity.${className};
  
public class ${className}Aggregate {  
    private final ${className} entity;  
  
    public ${className}Aggregate(${className} entity) {  
        this.entity = entity;  
    }  
  
    public ${className} getEntity() {  
        return this.entity;  
    } 

	public static ${className}Aggregate factory(${className} entity) {  
	    return new ${className}Aggregate(entity);  
	}
}`;
%>