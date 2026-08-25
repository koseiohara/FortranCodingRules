# Fortran Coding and Architecture Standard

本書は、Fortranによる科学技術計算programを新規作成または全面改修する際のarchitecture、module構成、
procedure構成、data ownership、命名、宣言、formatting、numerical operation、I/Oおよびexternal library利用を
規定する。

要求仕様に本書と矛盾する明示的な指定がある場合は、要求仕様を優先する。

本書に適合するsourceは、数値的に正しいだけでなく、architecture、責務分離、ownership、API、命名、宣言、
indentation、continuation、I/O等についても本規約を満たさなければならない。


## Language Level

sourceはstandard-conforming free-form Fortranとする。

Fortran 2008をbaselineとする。

Fortran 2008で定義されている任意の機能を使用可能とする。

より新しいstandardだけに存在する機能を使用する場合は、その機能が要求仕様またはalgorithm上必要であることを
確認する。

compiler固有のlanguage extensionへ依存してはならない。

C-preprocessorによるcompile-time configurationは許可する。


## Source Files

Fortran sourceの拡張子は必ず`.F90`とする。

拡張子のuppercaseは、file basenameおよびmodule名のuppercase禁止規則の例外とする。

basenameにはuppercase letterを一切使用しない。

basenameはlowercaseとする。

複数語からなるbasenameはlower snake_caseとする。

moduleを収録するfileのbasenameはmodule名と完全に一致させる。

一つの`.F90` fileには必ず一つのmoduleまたは一つのmain programだけを置く。

main programのfile名は`main.F90`とする。

source encodingはUTF-8 without BOMとする。

line endingはLFとする。

tab characterは禁止する。

trailing whitespaceは禁止する。


## Maximum Line Length

source lineはcommentを含めて最大125 columnsとする。

column 126以降へ文字を置いてはならない。

URLそのものをcharacter literalまたはcommentとして記載する必要がある場合だけ例外とする。

125 columnsを超えるstatementはcontinuationする。

procedure callについては、line lengthよりargument-count ruleを優先する。


## Main Program

`main`はapplication-level workflowだけを記述する。

`main`から見えてよいのは、大きなapplication phaseだけである。

以下は`main`へ置いてよい。

- configuration read routine
- program-wide initialization routine
- application-level output preparation routine
- top-level scientific computation call
- program-wide cleanup
- completion output

以下は`main`へ置いてはならない。

- scientific calculation loop
- statistical loop
- spatial-operation detail
- calculation-specific workspace
- calculation-specific large array
- calculation-specific allocation/deallocation
- calculation-specific special-case branch
- lower-level computational stageの羅列

一つのscientific computationを構成するlower-level routinesを、`main`から複数直接呼ばない。

一方、configuration readからcleanupまでapplication全体を一つの巨大な`run_*` procedureへ隠してはならない。


## Top-level Computational Procedure

一つのscientific computationには、一つのtop-level computational procedureを設ける。

`main`からscientific computationを開始するときは、原則としてこのprocedure一つを呼ぶ。

このprocedureがcalculation scopeに属する以下を所有する。

- main scientific arrays
- intermediate arrays
- statistical arrays
- final output arrays
- long-lived calculation workspace
- allocation
- deallocation
- lower-level operation calls
- calculationに属するbinary output

なお、top-level computational procedureを異なる条件で複数回呼ぶ実装では、mainでこのprocedureを複数回直接呼ぶことが許可される。

calculation-specific arrayを`main`に所有させてはならない。


## Module Responsibility

moduleはscientific responsibilityまたはfunctional responsibilityごとに分割する。

file lengthやprocedure countをmodule分割基準にしてはならない。

独立したscientific operationは独立moduleにする。

あるoperationが一箇所からしか呼ばれないという理由だけで、caller moduleのprivate helperへ埋め込んではならない。

独立したphysical、spatial、mathematical operationをstatistics moduleの内部helperにしてはならない。

一方、一つのoperationを構成する数statementをmoduleへ機械的に分割してはならない。


## Module-placement Tie Break

operationの配置に迷った場合は、以下を上から順に適用する。

1. 独立したscientificまたはphysical operationなら独立moduleとする。
1. external-library dependencyを局所化する責務を持つなら、そのoperation用moduleとする。
1. 上位layerへ見せるべきでないspecial-case branchを所有するなら、そのoperation用moduleとする。
1. 複数top-level computationsで再利用可能なら独立moduleとする。
1. 一つのtop-level computationだけに属するaggregation/statistical stageなら、そのcomputation moduleのprivate procedureとする。
1. 一つのprocedure内だけで意味を持つdetailならprocedure localとする。

procedureを短くすること自体を分割理由にしてはならない。


## Module Names

module名にはuppercase letterを一切使用しない。

module名はlowercaseとする。

複数語ではlower snake_caseとする。

module名は責務を表すnounまたはnoun phraseとする。

CamelCaseおよびPascalCaseは禁止する。


## Module Visibility

通常のfunctional/scientific moduleはdefault `private`とする。

`private`を明示する。

外部から必要なsymbolだけを`public`とする。

`public`では必ず`::`を使用する。

helper procedure、internal parameter、workspaceをpublicにしない。

public APIは最小限とする。

program-wide configuration/metadataを提供すること自体が責務であるglobal-state moduleについては、多数のmetadataを
共有する必要がある場合、その責務に適したaccessibilityを設定する。


## Module Layout

通常moduleの内部順序を以下に固定する。

1. blank line 2
1. `module`
1. module-scope `use`
1. blank line 1
1. `implicit none`
1. blank line 1
1. `private`
1. `public`
1. blank line 1
1. named constants
1. derived-type definitions
1. module variables
1. generic/interface blocks
1. blank line 1
1. `contains`
1. blank line 2
1. primary public procedure
1. private procedures in top-down call order
1. `end module <name>`
1. blank line 2

存在しないcategoryは省略する。


## Procedure Order

module内のproceduresはtop-down call orderで配置する。

順序は以下とする。

1. primary public entry point
1. primary entry pointから直接呼ばれるprivate procedures
1. さらにその下位のprivate procedures
1. pure utility functions

同levelの場合は実行順に置く。

alphabetical orderにはしない。


## `use` Statements

すべての`use` statementで必ず`only`を使用する。

対象がintrinsic module、external library module、project moduleのいずれであっても例外を設けない。

使用するsymbolをすべて明示的に列挙する。

`only`を省略した`use` statementは禁止する。

使用するsymbolが多いこと、`only` listが長いこと、記述が煩雑になることは、省略理由として認めない。

必要なsymbolが多い場合はcontinuationして列挙する。

未使用symbolを`only` listへ含めてはならない。


## `use` Scope

一つのprocedureだけが必要とするdependencyはprocedure内部で`use`する。

module-level declarationまたは複数proceduresが必要とするdependencyだけをmodule scopeへ置く。

procedure-local dependencyを便宜上module scopeへ引き上げてはならない。


## `use` Ordering

`use` statementsは以下の順序で並べる。

1. intrinsic modules
1. external libraries
1. project modules

group間にblank lineを入れない。

同group内はsemantic dependency順とする。

alphabetical sortを必須としない。

連続する`use` statementsでは`, only`のcolumnを揃える。


## Long `only` Lists

125 columns以内に収まる場合は1行にする。

125 columnsを超える場合はcontinuationする。

continuation元とcontinuation先の両方に`&`を置く。

continuation lineの最初のimported symbolを、前行の最初のimported symbolと対応する位置へ揃える。

一行一symbolにはしない。

symbol順はsemantic groupingを優先する。


## Intrinsic Modules

intrinsic moduleには`use, intrinsic ::`を使用する。

external/project moduleの`use`では`use ::`を使用する。

renameを行う場合も`only` list内で明示する。

rename operator `=>`の前後にspaceを置かない。


## `implicit none`

moduleには`implicit none`を一回記述する。

main programにも`implicit none`を記述する。

module procedure内には、module scopeからhost associationされる`implicit none`を重複記述しない。

`implicit none(type, external)`は使用しない。


## Global State

global stateとして保持してよいものはprogram-wide configurationまたはmetadataに限定する。

許可するもの:

- namelist input
- dimensions
- coordinates
- datetime metadata
- ctl metadata
- record metadata
- derived configuration
- program-wide constants

禁止するもの:

- calculation workspace
- temporary scientific fields
- intermediate statistical arrays
- calculation-specific outputs

initialization後readonlyとして扱うmetadataには`protected`を付ける。

namelist等から外部設定するvariableには必要性なく`protected`を付けない。


## Global Metadata Grouping

global declarationsはsemantic groupごとにsection分けする。

section commentは`!! -------------- <foo> -------------- !!`の形式を使用する。

section commentの前にはblank lineを1行置く。


## Initialization

program-wide initializationには一つのpublic `init` entry pointを設ける。

`init`内部からsemantic groupごとのprivate `init_*` procedureを呼ぶ。

一つの`init_*` procedureは一つのmetadata/configuration responsibilityを持つ。

derived configurationはcore calculation開始前に一度計算する。


## Data Ownership

major dataにはownerを一つ定める。

ownershipの基本は以下とする。

- program-wide configuration/metadata → global-state module
- calculation-wide arrays → top-level computational procedure
- procedure-local workspace → 当該procedure
- independent operationのinternal state → 当該operation module/procedure

ownerが曖昧なarrayを作らない。


## Allocation Ownership

allocateしたscopeが原則deallocateも行う。

通常のcomputational procedureではdummy argumentへ`allocatable`を付けない。

`allocatable` dummyを許可するのは、allocation自体がresponsibilityであるinitialization routine等だけとする。

lower-level computational procedureにはallocation済みassumed-shape arrayを渡す。

procedure-local workspaceはlocal allocatableとしてよい。

小さく、entry時にsizeが確定するworkspaceはautomatic arrayとしてよい。

loop内で不要なallocate/deallocateを行わない。


## Arrays

array dummyはassumed-shapeを基本とする。

`contiguous`属性の付与を積極的に検討する。

shapeはscientific dimensionsを可能な限り直接表す。

不必要なflatteningは禁止する。

major arrayでdimensionの意味が名前から明らかでない場合のみdimension commentを付ける。

配列の参照には、必ずarray sectionを明記する。
すなわち、`a=b`や`a(:)=1`のような記法を配列に対して例外なく禁止し、必ず`a(1:n)=b(1:n)`や`a(1:n)=1`のように記載する。
これにより、配列とスカラの区別がしやすくなるとともに、allocatable arrayに対する自動再割り付けの使用が禁止される。


## Dimension Comments

dimension commentはdeclaration末尾へ`!! [foo,bar]`の形式で置く。

dimension間のcomma後にspaceを置かない。

shapeが明白な場合は付けない。


## Dummy Arguments

適用可能なdummy argumentには必ず`intent`を付ける。

通常array dummyへ`allocatable`を付けない。

array storageの連続性をprocedureが要求して問題ないと判断できる場合は`contiguous`を付ける。


## Procedure Responsibility

一つのprocedureは一つの明確なoperationを担当する。

以下の独立責務を無秩序に混在させない。

- metadata interpretation
- physical operation
- spatial operation
- statistical aggregation
- validation
- control-file generation
- binary output

一方、一つのoperationを数statementごとにprocedure分割してはならない。


## Wrappers

wrapperは、上位layerからimplementation detailを隠し、一つのsemantic operationを提供する場合だけ作る。

input shapeやspecial caseによるimplementation branchは、そのoperationを所有するwrapper内に置く。

caller側へimplementation-specific branchを漏らさない。

以下は禁止する。

- constant getter
- single-statement meaningless wrapper
- procedure数を増やすだけのhelper


## Special Cases

special caseは、その意味を所有するmodule/procedureへ置く。

caller's responsibilityと無関係なspecial caseをcallerへ漏らさない。

special caseによってtop-level calculation structureを汚さない。


## Reusable Algorithms

同一algorithmを複数quantityへ適用できる場合は一つのgeneric operation procedureへまとめる。

quantityごとに同じimplementationを複製しない。

ただしscientific definitionが異なるものを、式が似ているという理由だけで共通化しない。


## `pure`

`pure`として成立するprocedureには可能な限り`pure`を付ける。

procedure設計時から、side effectが不要なoperationはpureとして成立する形を優先する。

特に以下はpure applicabilityを必ず確認する。

- numerical transformations
- index calculations
- mapping functions
- local statistical helpers
- local array/scalar calculations

pure化だけを目的にresponsibilityを歪めたりmeaningless wrapperを追加したりしてはならない。


## `recursive`

`recursive` procedureは原則禁止する。

非再帰implementationを使用する。

recursiveの方が短い、数学的定義がrecursiveである、prototypeがrecursiveである、という理由では使用しない。

要求仕様またはalgorithm上、本質的に再帰が必要な場合だけ例外を許可する。


## `elemental`

semantic operationがscalar elemental operationとして自然な場合は使用可能とする。

elemental化のためにinterfaceやalgorithmを歪めてはならない。


## `where` and `do concurrent`

`where`は禁止しない。

`do concurrent`は禁止しない。

通常`do`、array syntax、`where`、`do concurrent`から、semanticsとdata dependencyに合うものを選ぶ。

dependencyが存在するloopを`do concurrent`へしてはならない。

不要なlarge temporaryが生じるarray syntax/`where`はmemory costを確認する。


## Naming General

user-defined identifiersはlower snake_caseを基本とする。

短いscientific dimension/index/kind namesは例外として短い名称を使用してよい。

一般的なsoftware-engineering styleを理由として名前を不必要に長くしない。

独自のproject-wide vocabularyを勝手に作らない。


## Procedure Naming

procedure名はoperationを表す。

基本prefixは以下とする。

- initialization → `init_*`
- computation → `compute_*`
- input → `read_*`
- output → `write_*`
- accessor → `get_*`
- cleanup → `free`

より具体的な動詞が自然な場合はそれを使用してよい。


## Index Naming

x, y, and z indexには原則としてそれぞれ`i`, `j`, and `k`を使用する。
ただし、これに従うためにloop counter変数の宣言が乱立してしまう場合、loop counterとして宣言済みの変数を積極的に使い回す。

time indexには原則`t`を使用する。
ただし、`t`を別の変数として使用済みの場合は`tt`を使用する。

minute, hour, day, month, and year indexには原則`minute`, `hour`/`h`, `day`/`d`, `month`/`m`, and `year`/`y`をそれぞれ使用する。

generic loop indexには`i`, `j`, `k`等を使用してよい。


## Generic Names

inputのscientific meaningを意図的に限定しないgeneric operationでは、semantic-neutralなnamesを使用してよい。

同種のgeneric arraysには`arr1`, `arr2`, ...を使用してよい。

number suffixを一律禁止しない。

generic interfaceへ存在しないscientific semanticsをnameだけで付加してはならない。

明確なscientific quantityを持つdataにはquantityを表すnameを使用する。


## Temporary Names

generic workspaceには`work`を使用する。

意味を限定したworkspaceには`work_*`を使用してよい。

`tmp`や`tmp_*`は、温度に関する変数との混同を避けるため禁止する。

`data1`, `arr1`等を一律禁止しない。

名前のspecificityはinterfaceのsemantic specificityと一致させる。


## Kind Names

ソースコード全体で使用するworking real kindは`rk`とする。

ソースコード全体で使用するworking integer kindは`ik`とする。

同じ意味に別のkind aliasを増やさない。

kind parameterは、`iso_fortran_env`から要求仕様で定めたkindをrenameして宣言する。

working precisionが32-bit realの場合は`real32`を`rk`へrenameする。

working precisionが64-bit realの場合は`real64`を`rk`へrenameする。

ただし、一部の用途・一部のスコープのみで特別に異なるkind parameterの変数を使用する場合は、個別に適切な名前により宣言する。

kindの選択は要求仕様、I/O format、numerical requirementから決定し、実装者が任意に変更してはならない。

プログラム全体で共通のkind parameterを使用する場合には、global-state module内で一回宣言し、以後はそのmoduleから
kind parameterを参照する。

## Precision

default realへ依存しない。

real variablesはworking kindを明示する。

rangeが大きくなり得るrecord/time/sample-related integerには適切なinteger kindを使用する。

kind conversionは明示する。

mixed-kind arithmeticを不要にimplicit conversionへ任せない。


## Literals

working-precision real literalには`_rk`、integer literalには`_ik` suffixを付ける。

literal zero/oneもkind-sensitive calculationではkindを明示する。

ただし、integerに限り、表現範囲がdefault型で十分と判断される場合にはkind parameterの明示は不要とする。


## Procedure-local Layout

procedure内部の順序を以下に固定する。

1. procedure statement
1. procedure-local `use`
1. blank line 1
1. dummy declarations
1. blank line 1
1. derived-type locals
1. allocatable locals
1. automatic/fixed arrays
1. scalar real locals
1. scalar integer locals
1. scalar logical locals
1. blank line 1
1. executable statements
1. end statement

categoryが空なら省略する。

dummy blockとlocal-variable blockの間にはblank lineを1行置く。


## Declaration Entity Count

一つのdeclaration statementには原則として一つのentityだけを宣言する。

array shapeはentity側へ記述する。

`dimension` attributeは使用しない。


## Type Syntax

real kindは`real(rk)`とする。

`real(kind=rk)`は使用しない。

character assumed lengthは`character(*)`とする。

`character(len=*)`は使用しない。

derived typeは`type(foo)`の形式とする。


## Declaration Attributes

同一declaration groupではattributeの順序を統一する。

dummy argumentでは`intent(...)`を明示する。

`contiguous`が必要な場合は明示する。

`allocatable`、`protected`等は、そのentityのownershipまたはaccessibility上必要な場合だけ使用する。

同一group内でattribute順を揺らさない。


## Declaration Alignment

blank lineで区切られていない一つのdeclaration group全体をalignment unitとする。

group内では以下を揃える。

- type field
- comma column
- attribute fields
- `::`
- entity start column

最大type/attribute lengthへ合わせてASCII spacesでpaddingする。

別group間ではalignmentしない。


## Keyword Case

Fortran language keywordsはlowercaseで書く。

intrinsic procedure namesもlowercaseで書く。

user-defined identifiersもlowercaseを基本とする。

logical literalおよびlogical operatorはproject内で統一された表記を使用する。
`.TRUE.`, `.FALSE.`, `.AND.`, `.OR.`, and `.NOT.`はuppercaseで表記する。

preprocessor macrosはuppercaseを使用する。


## Relational Operators

relational operatorはsymbol formを使用する。

使用する。

- `==`
- `/=`
- `<`
- `<=`
- `>`
- `>=`

使用しない。

- `.eq.`
- `.ne.`
- `.lt.`
- `.le.`
- `.gt.`
- `.ge.`


## Basic Spacing

assignment `=`の前後には1 space置く。

comparison operatorの前後には1 space置く。

logical operatorの前後には1 space置く。

commaの前にはspaceを置かない。

procedure argument listのcomma後には1 space置く。

array subscriptsおよびarray shapeのcomma後にはspaceを置かない。

opening parenthesis直後にはspaceを置かない。

closing parenthesis直前にはspaceを置かない。

array-section colon周辺にはspaceを置かない。

semicolonによる複数statement記述は禁止する。


## Arithmetic Spacing

通常のbinary arithmetic operatorの前後にはspaceを置く。

ただし、coefficient multiplication、dimension-size expression、record-size expression等で一つのcompact termとして
扱う積はspaceを省略してよい。

subscript/index calculationで一つのcompact index termを形成する`+`/`-`もspaceを省略してよい。

同じexpression levelではstyleを混在させない。


## Unary Operators

unary `+`および`-`とoperandの間にはspaceを置かない。


## String Literals

character literalはsingle quoteを標準とする。

single quoteをliteral内部に多数含むため可読性が大きく低下する場合だけdouble quoteを許可する。


## Component Selector

derived-type componentおよびtype-bound procedure accessでは`%`の前後に1 space置く。


## Indentation

indentation unitはASCII spaces 4文字とする。

tabは禁止する。

nestが一段増えるごとに4 spaces増やす。

continuation alignmentはblock indentationとは別規則で決める。


## `contains` statement

`contains`文は、直前の行と同じ深さのindentに記述する。

`contains`文の次の行は、`contains`と同じ深さのindentに記述する。


## `if` Syntax

block formは`if (condition) then`とする。

keywordとopening parenthesisの間に1 space置く。

parenthesis内側にはspaceを置かない。

`else if`は二語で記述する。

`elseif`は使用しない。

block endは`endif`とする。

single-line `if`は例外なく禁止する。


## `do` Syntax

loop headerではkeyword後に1 space置く。

loop variable assignmentの`=`前後に1 space置く。

comma後に1 space置く。

block endは`enddo`とする。

construct nameは不必要な場合は使用しない。


## `select`

`select case`, `select type`等はstandard two-word syntaxを使用する。

closing statementはstandard spaced formを使用する。

construct nameは原則使用しない。


## Procedure End Statements

procedure、function、module、programのend statementには必ずnameを付ける。

anonymous end statementを使用しない。


## Function Form

functionでは必ず`result(...)`を使用する。

function statementにreturn typeを置かない。

result variableはdeclaration blockで宣言する。


## Procedure Call Breaking

`call` statementについて、actual argumentsが0個または1個の場合は125 columns以内なら一行にする。

actual argumentsが2個以上の場合は、行長にかかわらず必ずmultilineにする。

2 arguments以上の`call`を一行で記述してはならない。


## Canonical Multiline Call Layout

multiline `call`では第1argumentをprocedure nameと同じ行に置く。

opening parenthesis直後で改行してはならない。

第2argument以降はcontinuation lineへ置く。

continuation元とcontinuation先の両方に`&`を置く。

argument本体を第1argument開始columnへ縦に揃える。

non-final argumentとfinal argumentでは、argument本体の後ろをspaceでpaddingし、次のsuffixを同じcolumnsへ配置する。

- non-final argument: `, &`
- final argument: `  )`

したがって、`,`の直前に置かれるspaceはargument separatorのspacingではなく、`, &`と`  )`をcolumn alignmentするためのpaddingである。

canonical layoutは次の形式とする。

```fortran
call foo(input_a , &  !! IN
       & input_b , &  !! IN
       & output_c  )  !! OUT
```

argument長が異なる場合は、最長のargumentに合わせてargument本体の後ろをspaceでpaddingする。

```fortran
call foo(x                 , &  !! IN
       & long_variable_name, &  !! IN
       & result              )  !! OUT
```

すなわち、non-final argumentでは`,`のcolumnおよび`&`のcolumnをそれぞれ揃え、final argumentでは`,`に対応するcolumnをspaceとし、`&`に対応するcolumnへ`)`を置く。

spaceを`_`で表した場合、suffix部分は次の配置になる。

```text
,_&
__)
```

closing parenthesisだけを独立行へ置いてはならない。

multiline scientific/functional `call`では、`&`または`)`の後に2 spaces置き、intent属性を次のように記載する。

- `!! IN`
- `!! OUT`
- `!! INOUT`

procedureの場合は次のように記載する。

- `!! PROCEDURE`


## Type-bound Calls

type-bound `call`にもargument-count breaking ruleを適用する。

0または1 argumentは125-column ruleに従う。

2 arguments以上は必ずmultilineとする。


## Function References

argument-countによる強制改行規則は`call` statementに適用する。

function referenceには自動的に適用しない。

function expressionは125-column ruleおよびexpression-continuation ruleに従う。


## Constructors

derived-type constructorが125 columnsを超える場合はmultilineとする。

multiline constructorでは第1keyword argumentをopening parenthesisと同じ行へ置く。

後続linesはprocedure callと同じcontinuation alignmentを用いる。

keyword argumentsを縦に配置する場合は対応するlogical elementを揃える。

closing parenthesisはfinal argument lineへ置く。


## Keyword Argument Spacing

keyword argumentでは、keywordの一文字目と`=`のcolumnをそれぞれ揃える。


## Expression Continuation

長いexpressionをcontinuationする場合、binary operatorは継続元lineの末尾側へ置く。

continuation line冒頭にoperatorを置かない。

continuation元とcontinuation先の両方に`&`を置く。

expression structureに対応してalignmentする。


## Blank Lines

blank lineはlogical structureを示す。

blank lineの連続数は以下とする。

- module-level `contains`直後 → 2 blank lines
- procedure間 → 2 blank lines
- その他のlogical boundary → 1 blank line
- それ以外 → 0 blank lines

3行以上連続するblank lineは禁止する。


## Required Blank-line Boundaries

以下の境界にはblank lineを1行置く。

- module-scope `use` blockの後
- `implicit none`の後
- `private/public` blockの後
- module declarationsの後
- procedure-local `use` blockの後
- dummy declarationsの後
- local declarationsの後
- allocation groupの後
- major scientific stagesの間
- calculationとoutputの間
- outputとcleanupの間

密接な同一stage内にはblank lineを入れない。


## Assignment Alignment

同一purposeのassignmentが2行以上連続する場合は、`=` columnを極力揃える。

alignmentによって極端に大きな空白が生じる場合は、semantic groupingを優先する。


## Comments

通常source commentには`!!`を使用する。

`!!`の後に1 space置く。

source commentsは英語で記述することとし、その他の言語の使用を例外なく禁止する。

statementをそのまま自然言語化したcommentは禁止する。


## Inline Comments

inline commentは以下を主用途とする。

- argument direction
- array dimension
- non-obvious implementation constraint
- non-obvious scientific meaning

通常statementの逐語説明には使用しない。


## Namelist

namelistにはuserが決定する必要があるparameterだけを置く。

metadata sourceから取得可能な値を重複入力させない。

read前にdefault valueを明示的に設定する。

read後にcheckerを呼ぶ。

invalid configurationをsilent correctionしてはならない。


## Metadata Source of Truth

ctlに存在するmetadataはctlをsource of truthとする。

ctlから取得可能なら以下をnamelistへ要求しない。

- grid dimensions
- coordinate values
- time-axis origin
- time increment
- variable definitions
- binary options
- record-layout metadata


## Error Handling

fatal errorではstandard error unitへ出力する。
error unitは必ず次のように宣言されなければならない`use, intrinsic :: iso_fortran_env, only : err=>error_unit`。

error blockの最初のmessageは`<ERROR STOP>`とする。

その後、具体的messageを書く。

fatal terminationにはupper caseにて`ERROR STOP`を使用する。

plain `stop`は使用しない。

central error wrapperは作らない。


## I/O

standard inputには`iso_fortran_env`の`input_unit`を`iunit`へrenameして使用する。

standard outputには`iso_fortran_env`の`output_unit`を`ounit`へrenameして使用する。

同一scopeでstandard input、standard output、standard errorのうち複数を使用する場合は、一つの`use, intrinsic :: iso_fortran_env, only : ...` statementへまとめる。

Fortran I/Oではinline format stringを使用する。

numeric statement-label formatは禁止する。

external-library file abstractionがある場合は、そのAPIの仕様を確認したうえで使用する。


## Binary I/O

出力ファイルフォーマットは、原則として`EDAT_BinIO`で出力可能なものを使用する。

binary I/O callの単位はlogical data recordと一致させる。

一つのlogical recordをscalar writesへ不必要に分割してはならない。

recordに関する演算をすることを目的として宣言されるinteger変数は、例外なく`int64`またはそれ以上の表現範囲を持つ型を使用する。


## Direct-access `RECL`

`RECL`の単位は必ずbyteとする。
コンパイラが異なる単位をデフォルトとしている場合には、コンパイラオプションによってbyteでの取り扱いを強制する。


## Output Packing

以下を完全に一致させる。

- work-array packing order
- binary record order
- variable metadata order
- ctl variable order

dynamic output countは同一configurationから式で導出する。

過大なfixed work bufferを使用しない。


## File-object Lifetime

file objectのconstruction/openとcloseは可能な限り同一procedureで完結させる。

file objectをprocedure間で不必要に持ち回らない。


## Numerical Accuracy

numerical correctnessとprecisionを計算速度とmemory usageに対して優先する。

existing high-precision routineがある場合、単純intrinsic implementationへ劣化させない。
特に、サイズが4以上のreal型配列要素の総和にはbuilt-in functionの`sum()`の使用を禁止し、EDATがもつ`sum_hp()`を必ず使用する。

integration、reduction、statisticsではlibrary implementationのmathematical definitionを確認する。

以下を必ず確認する。

- normalization
- denominator
- sample/population definition
- returned statistic
- confidence semantics
- kind
- status behavior


## External Library Policy

library APIを名前から推測してはならない。

README、documentation、interface、source implementationを確認する。

APIの説明または使用例を作る場合も、確認済みの仕様だけを使用する。

確認せずにargument order、return semantics、normalization、kind、side effect等を推測してはならない。


## EDAT

Repository:

https://github.com/koseiohara/EDAT

独自numerical/scientific routineを書く前に確認する。

少なくとも以下のcategoryでは最初にEDATを探索する。

- high-precision reduction
- statistics
- meteorological operations
- spatial integration
- differentiation
- binary I/O


## ctlget

Repository:

https://github.com/koseiohara/ctlget

GrADS control-file metadataの取得に使用する。

ctl metadataを独自parserまたはnamelist duplicationで代替しない。


## AutoCtl

Repository:

https://github.com/koseiohara/AutoCtl

GrADS control-file generationに使用可能とする。

独自にctl writerを新規実装する前にAPIを確認する。


## datetime-fortran

Repository:

https://github.com/wavebitscientific/datetime-fortran

datetimeおよびcalendar arithmeticに使用する。

leap year、month length、date increment、datetime difference等を独自実装しない。


## Performance

loop invariantはloop外で計算する。

core loop内で以下を不必要に行わない。

- allocation/deallocation
- file open/close
- metadata parsing
- constant/configuration calculation
- repeated validation

performanceのためにresponsibility boundaryを壊さない。


## Generic Interfaces

同一semantic operationのkind/type variationを一つのnameで扱う場合にgeneric interfaceを使用する。

unrelated operationsをgenericにまとめない。


## Derived Types

複数dataが一つのsemantic objectを形成する場合にderived typeを使用してよい。

argument数削減だけを目的にderived typeを作らない。

objectとoperation ownershipが自然な場合にtype-bound procedureを使用してよい。


## Preprocessor

preprocessor directiveはcolumn 1から開始する。

compile-time configurationにのみ使用する。

precision selectionやexternal-feature enable/disable等に使用してよい。

normal runtime algorithm branchをpreprocessorへ移さない。
