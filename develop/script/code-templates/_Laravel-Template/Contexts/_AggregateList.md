<%*
const { className } = tp.user.data;

tR += `<?php

namespace App\\Contexts\\Domain\\Collection;

use App\\Contexts\\Domain\\Aggregate\\${className}Aggregate;
use App\\Infrastructure\\Models\\${className};
use Illuminate\\Support\\Collection;

class ${className}AggregateList
{
    private readonly Collection $aggregates;

    public function __construct(Collection $aggregates)
    {
        $this->aggregates = $aggregates;
    }

    public function getAggregates(): Collection
    {
        return $this->aggregates;
    }

    public function findLast(): ?${className}Aggregate
    {
        return $this->aggregates->last();
    }

    public function isEmpty(): bool
    {
        return $this->aggregates->isEmpty();
    }

    public static function factory(?iterable $models): self
    {
        $collection = collect($models ?? [])
            ->map(fn(${className} $model) => new ${className}Aggregate($model));

        return new self($collection);
    }
}`;
%>
