# Plan9BasicEngine

Núcleo do interpretador **Plan9Basic** e a biblioteca padrão compartilhada
pelos projetos que o hospedam. Delphi / FireMonkey, MIT.

Este repositório existe para que exista **uma única cópia** deste código. Ele é
consumido como submódulo por:

| Projeto | Papel |
|---|---|
| [Plan9Basic](https://github.com/AndreMurtaX/Plan9Basic) | IDE e ambiente de desenvolvimento |
| [Plan9BasicAppletRunner](https://github.com/AndreMurtaX/Plan9BasicAppletRunner) | executor de applets distribuídos |

## Conteúdo

```
basic.pas               TBasicEngine — fachada do engine para a aplicação hospedeira
lexer.pas               tokenizador
parser.pas              parser e gerador de código intermediário/assembly
exec.pas                máquina de pilha que executa o assembly
UnitUtils.pas           utilitários compartilhados
utils/UnitGC.pas        coletor de lixo dos objetos não-visuais
utils/HandleRegistry.pas validação de handles sem dereferenciar ponteiro do programa
Libs/                   biblioteca padrão (Array, Str, Num, DateTime, Json, Http, Zip…)
Libs/GUI/TimerLib.pas   timers — dependência de exec.pas
Libs/AI/                cliente de IA e motor RAG
```

## Pipeline

```
fonte BASIC
   ↓  lexer.pas          tokenização
   ↓  parser.pas         validação sintática → código intermediário postfix
   ↓  ProcessPostfixCode geração de assembly
   ↓  exec.pas           máquina de pilha executa
saída
```

## Como usar

Adicione como submódulo e aponte o `.dpr` para os caminhos dentro dele:

```bash
git submodule add https://github.com/AndreMurtaX/Plan9BasicEngine.git engine
```

```pascal
uses
  basic in 'engine\basic.pas',
  exec in 'engine\exec.pas',
  lexer in 'engine\lexer.pas',
  parser in 'engine\parser.pas',
  UnitUtils in 'engine\UnitUtils.pas',
  UnitGC in 'engine\utils\UnitGC.pas',
  HandleRegistry in 'engine\utils\HandleRegistry.pas',
  StdLib in 'engine\Libs\StdLib.pas',
  // …
```

O hospedeiro cria o `TBasicEngine`, registra as bibliotecas que quiser expor e
executa:

```pascal
GC := TGarbageCollector.Create();
Engine := TBasicEngine.Create();
StdLib.RegisterStdFuncs(Engine.Functions);
NumLib.RegisterNumFuncs(Engine.Functions);
// …
if Engine.Compile(Source) = 0 then
  Engine.ExecuteProgram(Output);
```

Numa execução sem interface, ligue `UnitGC.SkipProcessMessages := True` para o
engine não tentar bombear mensagens.

## Modelo de extensão

Funções nativas são registradas por assinatura em string: `nome@parâmetros`,
onde o sufixo do nome indica o tipo de retorno (`$` string, `#` ponteiro,
nenhum = número) e cada parâmetro é `n`, `$` ou `#`.

```pascal
FnData.Entry := n_abs;   Lib.Add('abs@n', FnData);      // abs(número)
FnData.Entry := s_left;  Lib.Add('left$@$n', FnData);   // left$(texto, n)
```

## Testes

A suíte automatizada vive no repositório do IDE, em
[Plan9Basic/tests](https://github.com/AndreMurtaX/Plan9Basic/tree/main/tests):
um runner headless que compila e executa programas `.bas` e confere asserções.
Mudanças neste repositório devem ser validadas por ela.
