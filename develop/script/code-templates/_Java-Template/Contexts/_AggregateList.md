<%*
const { className } = tp.user.data;

tR += `package com.example.api.contexts.domain.collection.aggregate;  
  
import com.example.api.contexts.domain.aggregate.${className}Aggregate;  
  
import java.util.Arrays;  
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;
  
public class ${className}AggregateList {  
    private final List<${className}Aggregate> aggregates;  
  
    public ${className}AggregateList(List<${className}Aggregate> aggregates) {  
        this.aggregates = aggregates;  
    }

	public List<${className}Aggregate> getAggregates() {
		return this.aggregates;
	}
  
    public ${className}Aggregate findLast() {  
        return this.aggregates.get(this.aggregates.size() - 1);  
    }
  
	public boolean isEmpty() {  
	    return this.aggregates.isEmpty();  
	}

	public static ${className}AggregateList factory(List<${className}> entities) {
		if(Objects.isNull(entities)) return new ${className}AggregateList(List.of());
	    return new ${className}AggregateList(  
	            entities.stream()  
	                    .map(${className}Aggregate::new)  
	                    .collect(Collectors.toList())  
	    );  
	}
}`;
%>