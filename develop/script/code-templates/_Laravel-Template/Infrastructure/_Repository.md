<%*
const { tableNameJp, className } = tp.user.data;

tR += `<?php

namespace App\\Infrastructure\\Repository;

use App\\Contexts\\Domain\\Aggregate\\${className}Aggregate;
use App\\Contexts\\Domain\\Collection\\${className}AggregateList;
use App\\Infrastructure\\Models\\${className};

/**
 * ${tableNameJp}
 */
class ${className}Repository
{
    /**
     * レコードの取得（全件抽出）
     */
    public function all(): ${className}AggregateList
    {
        return ${className}AggregateList::factory(${className}::all());
    }

    /**
     * レコードの取得（主キー抽出）
     */
    public function find(int $id): ${className}Aggregate
    {
        return ${className}Aggregate::factory(${className}::findOrFail($id));
    }

    /**
     * レコードの取得（主キー複数抽出）
     */
    public function finds(array $ids): ${className}AggregateList
    {
        return ${className}AggregateList::factory(${className}::whereIn('id', $ids)->get());
    }

    /**
     * 登録（主キー生成）
     */
    public function insertGeneratedKeys(${className}Aggregate $aggregate): void
    {
        $aggregate->getModel()->save();
    }

    /**
     * 更新
     */
    public function update(${className}Aggregate $aggregate): void
    {
        $aggregate->getModel()->save();
    }

    /**
     * 物理削除
     */
    public function delete(${className}Aggregate $aggregate): void
    {
        $aggregate->getModel()->delete();
    }
}`;
%>
