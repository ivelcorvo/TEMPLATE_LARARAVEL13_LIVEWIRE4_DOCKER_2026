# Livewire 4 — Guia de Início com Single File Components (SFC)

O Livewire 4 introduziu suporte nativo a **Single File Components (SFC)**: arquivos `.blade.php` que contêm tanto a classe PHP (backend) quanto o template HTML (frontend) no mesmo arquivo, sem precisar de nenhum pacote extra.

> **Nota sobre o Volt:** O pacote Volt ainda existe como uma opção de sintaxe alternativa para SFC, mas **não é necessário instalá-lo**. O Livewire 4 já oferece SFC nativamente.

---

## Passo 1 — Instalar o Livewire 4

```bash
docker compose exec app composer require livewire/livewire:^4.0
```

se o pc for uma carroça é melhor definir o time global do composer antes
```bash
docker compose exec app composer config process-timeout 0
```

---

## Passo 2 — Criar o Layout Base

Execute o comando abaixo para gerar o arquivo de layout padrão:

```bash
docker compose exec app php artisan livewire:layout
```

Isso cria `resources/views/layouts/app.blade.php`. O Livewire 4 busca esse layout por padrão com o nome `layouts::app`. Abra e edite para o seguinte:

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{ $title ?? config('app.name') }}</title>

    @vite(['resources/css/app.css', 'resources/js/app.js'])

    @livewireStyles
</head>
<body>

    {{-- Navbar, sidebar, etc. ficam aqui --}}

    {{ $slot }}  {{-- ← aqui cada Page Component é injetado --}}

    @livewireScripts
</body>
</html>
```

> **Por que `@livewireStyles` e `@livewireScripts`?**
> A documentação oficial do Livewire 4 inclui essas diretivas no layout padrão gerado pelo `livewire:layout`. Mantenha-as para garantir que os assets do Livewire sejam carregados corretamente.

---

## Passo 3 — Criar um Page Component (SFC)

Componentes usados como páginas inteiras são criados com o namespace `pages::`:

```bash
docker compose exec app php artisan make:livewire pages::dashboard
```

Isso cria o arquivo em `resources/views/pages/⚡dashboard.blade.php`.

Abra o arquivo e edite para o seguinte SFC completo — PHP e HTML no mesmo arquivo:

```blade
{{-- resources/views/pages/⚡dashboard.blade.php --}}

<?php

use Livewire\Attributes\Title;
use Livewire\Attributes\Layout;
use Livewire\Component;

new #[Title('Dashboard')]
    #[Layout('layouts::app')]   {{-- layout padrão; pode omitir se não customizar --}}
    class extends Component {

    public string $mensagem = 'Bem-vindo ao Dashboard!';
    public int $contador = 0;

    public function incrementar(): void
    {
        $this->contador++;
    }
}
?>

{{-- Template da página --}}
<div>
    <h1>{{ $mensagem }}</h1>
    <p>Contador: {{ $contador }}</p>
    <button wire:click="incrementar">+1</button>
</div>
```

> **Por que não há `render()` no SFC?**
> Quando PHP e template estão no mesmo arquivo `.blade.php`, o Livewire 4 infere o template automaticamente. O método `render()` só é necessário em componentes multi-file (classe `.php` separada do template).

---

## Passo 4 — Registrar a Rota

No Livewire 4, page components são roteados com `Route::livewire()`:

```php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Sem middleware — funciona sem usuário autenticado
Route::livewire('/dashboard', 'pages::dashboard');
```

Acesse em: `http://localhost:8080/dashboard`

Quando você implementar autenticação (Laravel Breeze, Fortify ou manual), adicione o middleware normalmente:

```php
// routes/web.php

use Illuminate\Support\Facades\Route;

// Rota simples protegida
Route::livewire('/dashboard', 'pages::dashboard')->middleware('auth');

// Rota com parâmetro
Route::livewire('/posts/{id}', 'pages::show-post');

// Múltiplas rotas protegidas em grupo
Route::middleware(['auth', 'verified'])->group(function () {
    Route::livewire('/perfil',     'pages::perfil');
    Route::livewire('/relatorios', 'pages::relatorios');
});
```

---

## Passo 5 — Criar um Componente Reutilizável

Componentes reutilizáveis (que não são páginas) são criados sem namespace:

```bash
docker compose exec app php artisan make:livewire contador
```

Isso cria `resources/views/components/⚡contador.blade.php`. Abra e edite para o seguinte SFC:

```blade
{{-- resources/views/components/⚡contador.blade.php --}}

<?php

use Livewire\Attributes\Locked;
use Livewire\Component;

new class extends Component {

    // Prop recebida do pai — protegida contra alteração pelo frontend
    #[Locked]
    public string $titulo = 'Contador';

    // Estado interno do componente
    public int $valor = 0;

    public function incrementar(): void
    {
        $this->valor++;
    }

    public function decrementar(): void
    {
        if ($this->valor > 0) {
            $this->valor--;
        }
    }
}
?>

<div>
    <h3>{{ $titulo }}</h3>
    <p>{{ $valor }}</p>

    <button wire:click="decrementar" wire:loading.attr="disabled">-</button>
    <button wire:click="incrementar" wire:loading.attr="disabled">+</button>

    {{-- Feedback visual enquanto o servidor responde --}}
    <span wire:loading>Atualizando...</span>
</div>
```

---

## Passo 6 — Usar o Componente dentro de uma Página

Dentro de qualquer template Blade (inclusive de outro componente Livewire):

```blade
{{-- trecho do template do dashboard --}}
<div>
    <h1>Dashboard</h1>

    {{-- Passando uma string literal --}}
    <livewire:contador titulo="Visitas hoje" />

    {{-- Passando uma variável dinâmica com :key para forçar re-mount --}}
    <livewire:contador :titulo="$minhaVariavel" :key="$minhaVariavel" />
</div>
```

> **Regra do `:key`:** Use sempre que o componente depende de dados dinâmicos do pai e precisa ser recriado quando esses dados mudam.

---

## Resumo — Estrutura de arquivos final

```
resources/views/
├── layouts/
│   └── app.blade.php              ← Passo 2: Layout base (gerado por livewire:layout)
├── pages/
│   └── ⚡dashboard.blade.php      ← Passo 3: Page Component (SFC) — namespace pages::
└── components/
    └── ⚡contador.blade.php        ← Passo 5: Componente reutilizável (SFC)

routes/
└── web.php                        ← Passo 4: Route::livewire()
```

---

## Regras de ouro para não errar

| O que fazer | Por quê |
|---|---|
| `#[Locked]` em props que o frontend não deve alterar | Segurança — qualquer `public` sem `Locked` pode ser alterado via request forjado |
| `wire:loading.attr="disabled"` nos botões | Evita double-submit enquanto o servidor processa |
| `:key` em componentes com dados dinâmicos | Garante re-mount correto quando os dados mudam |
| Nunca usar `@yield` no layout | Livewire 4 usa `$slot`, não herança de template |
| `Route::livewire()` para page components | É a forma oficial do Livewire 4 para rotear páginas |
| Omitir `render()` no SFC | O Livewire 4 infere o template automaticamente no mesmo arquivo |
| Usar `pages::` ao criar page components | Organiza os arquivos em `resources/views/pages/`, separando pages de componentes reutilizáveis |
| Lógica de negócio em métodos `public` isolados | Fica testável e o Livewire consegue chamar via `wire:click` |
