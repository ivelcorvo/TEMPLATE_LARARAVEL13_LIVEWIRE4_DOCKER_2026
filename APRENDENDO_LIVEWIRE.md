# Livewire 4 

Livewire 4 tem SFC nativo — você não precisa instalar o Volt. O Livewire v4 substitui o Volt. SFC é uma funcionalidade do próprio Livewire 4, sem precisar do pacote separado. O diagrama acima mostra como as peças se encaixam. GitHub

## Passo 1 — Instalar o Livewire 4

```bash
docker compose exec app composer require livewire/livewire:^4.0
```
se o pc for uma carroça é melhor definir o time global do composer antes
```bash
docker compose exec app composer config process-timeout 0
```

## Passo 2 — Criar o LayoutExecute 

o comando abaixo para gerar o arquivo de layout padrão do Livewire
```bash
docker compose exec app php artisan livewire:layout
```
Isso cria resources/views/layouts/app.blade.php.

abra e edite para o seguinte:
```bash
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

## Passo 3 — Criar um Page Component (Single File)

Ao criar componentes que serão usados como páginas inteiras, use o namespace pages:: para organizá-los em um diretório dedicado.

```bash
docker compose exec app php artisan make:livewire pages::dashboard
```

Isso cria um único arquivo em resources/views/pages/⚡dashboard.blade.php (o ⚡ é opcional e serve só para identificação visual no editor).
O arquivo gerado é o SFC completo — PHP e HTML no mesmo lugar:
```bash
{{-- resources/views/pages/dashboard.blade.php --}}

<?php

use Livewire\Attributes\Title;
use Livewire\Attributes\Layout;
use Livewire\Component;

new #[Title('Dashboard')]          // título da aba do browser
    #[Layout('layouts.app')]       // qual layout usar (padrão já é esse)
    class extends Component {

    public string $mensagem = 'Bem-vindo ao Dashboard!';
    public int $contador = 0;

    public function incrementar(): void
    {
        $this->contador++;
    }

    public function render(): \Illuminate\Contracts\View\View
    {
        return view('livewire.pages.dashboard'); // aponta pro template abaixo
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

## Passo 4 — Registrar a rota no web.php

Para rotear para um componente, use o método Route::livewire() no arquivo routes/web.php. Laravel
ss
```bash
// routes/web.php

use Illuminate\Support\Facades\Route;

// ✅ Sem middleware — funciona sem usuário
Route::livewire('/dashboard', 'pages::dashboard');

// ❌ NÃO use ainda — exige autenticação
// Route::livewire('/dashboard', 'pages::dashboard')->middleware('auth');
```

Acesse em: http://localhost:8080/dashboard
Quando você implementar o login (Laravel Breeze, Fortify, ou manual), aí sim você volta e adiciona o ->middleware('auth'). Não tem nenhuma outra dependência no guia que precise de usuário — layout, SFC, componente e rota funcionam todos sem autenticação.

quando tiver usuários
```bash
// routes/web.php

use Illuminate\Support\Facades\Route;

Route::livewire('/dashboard', 'pages::dashboard')
    ->middleware('auth');   // adicione middlewares normalmente

// Rota com parâmetro
Route::livewire('/usuarios/{id}', 'pages::usuario-detalhe');

// Múltiplas rotas protegidas
Route::middleware(['auth', 'verified'])->group(function () {
    Route::livewire('/perfil',     'pages::perfil');
    Route::livewire('/relatorios', 'pages::relatorios');
});
```

## Passo 5 — Criar um Componente Reutilizável

Componentes reutilizáveis (não são páginas) ficam em resources/views/livewire/:
```bash
docker compose exec app php artisan make:livewire contador
```

Cria resources/views/livewire/⚡contador.blade.php:
```bash
{{-- resources/views/livewire/contador.blade.php --}}

<?php

use Livewire\Attributes\Locked;
use Livewire\Component;

new class extends Component {

    // Prop recebida do pai — readonly do lado do frontend
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

    public function render(): \Illuminate\Contracts\View\View
    {
        return view('livewire.contador');
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
## Passo 6 — Usar o Componente dentro de uma Página

Dentro de qualquer template Blade (inclusive de outro Livewire):

```bash
{{-- dentro do template do dashboard --}}
<div>
    <h1>Dashboard</h1>

    {{-- Sintaxe de tag Blade --}}
    <livewire:contador titulo="Visitas hoje" />

    {{-- Ou com chave para forçar re-mount quando o ID mudar --}}
    <livewire:contador :titulo="$minhaVariavel" :key="$minhaVariavel" />
</div>
```

### Regra do key: Sempre use :key quando o componente depende de dados dinâmicos do pai e precisa ser recriado quando esses dados mudam.

# Resumo — Estrutura de arquivos final
```
resources/views/
├── layouts/
│   └── app.blade.php          ← Passo 2: Layout base
├── pages/
│   └── ⚡dashboard.blade.php   ← Passo 3: Page Component (SFC)
└── livewire/
    └── ⚡contador.blade.php     ← Passo 5: Componente reutilizável (SFC)

routes/
└── web.php                    ← Passo 4: Route::livewire()
```
# Regras de ouro para não errar

|O que fazer | Por quê |
|---|---|
| `#[Locked]`  em props que o frontend não deve alterar | Segurança — qualquer `public` sem `Locked` pode ser alterado via request forjado |
| `wire:loading.attr="disabled"` nos botões | Evita double-submit enquanto o servidor processa |
| `:key` em componentes com dados dinâmicos | Garante re-mount correto quando os dados mudam |
| Nunca usar `@yield` no layout | Livewire 4 usa `$slot`, não herança de template |
| `Route::livewire()` em vez de `Route::get()` + view | É a forma correta para page components |
| Lógica de negócio em métodos `public` isolados | Fica testável e o Livewire consegue chamar via `wire:click` |
