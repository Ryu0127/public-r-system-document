<%*
const { className } = tp.user.data;

tR += `<?php

namespace App\\Contexts\\Domain\\Aggregate;

use App\\Infrastructure\\Models\\${className};

class ${className}Aggregate
{
    private readonly ${className} $model;

    public function __construct(${className} $model)
    {
        $this->model = $model;
    }

    public function getModel(): ${className}
    {
        return $this->model;
    }

    public static function factory(${className} $model): self
    {
        return new self($model);
    }
}`;
%>
