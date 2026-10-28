# PHP5.x 7.x 升级到PHP8.2检查清单

版本：1.6｜核对日期：2026-10-29｜目标运行时：PHP 8.2。

用途：检查 PHP 5.x / 7.x 代码迁移到 8.2 时的语言、标准库、内置扩展、配置和格式兼容性。清单不包含具体业务系统、业务流程、项目案例或一般服务器故障。

收录边界：项目、框架或第三方库的专有函数名称、业务逻辑及自身实现缺陷不单列；只有能归因于 PHP 语言、内置 API/扩展或运行时版本变化的规则才纳入。自定义函数或方法触发这些规则时，只提炼通用规则，不保留项目专有名称和修法。PHP 规定的魔术方法、内置接口实现方法，以及统一适用的参数、继承、回调和错误处理规则仍属本清单范围。表中 f、C、m 等为语法示意占位符，不代表项目函数。

使用方法：确认原始 PHP 小版本，逐项填写“通过 / 待修 / 不适用”，不适用也注明理由。**移除**表示目标版本不能继续原用法；**弃用**表示 8.2 中通常尚可使用但产生弃用诊断；**变化**表示结果、错误级别或调用契约不同。先排语法/移除项，再查类型/签名/默认值，最后查静默行为差异。固定目录无法穷尽所有可选扩展，末尾官方索引用于补查实际使用的扩展。

## 1. 语法、关键字与代码格式

| 状态 / 编号 | 变化版本 | 搜索或检查对象 | 核对 / 处理要点 |
| --- | --- | --- | --- |
| [ ] SYN01 | 5.4 移除 | 调用时引用 `f(&$x)` | 引用写在函数声明；普通调用改为 `f($x)` |
| [ ] SYN02 | 7.0 移除 | `$x =& new C()` | 去除旧实例化引用语法，核对对象赋值语义 |
| [ ] SYN03 | 7.0 移除 | ASP 标签 `<% %>`、`<%= %>`、`<script language="php">` | 改 `<?php` / `<?=`；`<?` 短标签受配置控制，不能误写成 8.2 已全部移除 |
| [ ] SYN04 | 7.4 弃用 → 8.0 移除 | `$s{0}`、`$a{$k}` | 下标改 `[]`；不要改代码块、对象动态访问或合法插值花括号 |
| [ ] SYN05 | 7.4 弃用 → 8.0 非结合 | 无括号连续三元 `a?b:c?d:e` | 明确结合方式；保留旧行为通常须显式左结合，不能一律右嵌套 |
| [ ] SYN06 | 8.0 变化 | `.` 与 `+ - << >>` 混用 | 连接优先级降低；按预期加括号 |
| [ ] SYN07 | 8.0 移除 | `(real)`、`(unset)` | 前者用 `(float)`；后者按原结果语义重写，不混同 `unset()` |
| [ ] SYN08 | 7.2 警告 → 8.0 Error | `$a[key]`、未定义裸常量 | 字符串键加引号；真正常量修定义，不批量把所有标识符转字符串。`php -l` 通过不代表该键可用：最小用例须实际执行取值表达式，并以非空数组覆盖循环体，核对取值及其传入后续函数的参数未改变 |
| [ ] SYN09 | 7.0 起分批保留 | 类/接口/trait/函数/命名空间名称 | 检查 int/string/bool/float、void/iterable/object、fn、match/mixed、readonly/never 等；限制按具体命名位置核实 |
| [ ] SYN10 | 8.0 变化 | `#[` 开头的旧 # 注释 | 可能被解析为 Attribute；重新确认注释，不删除真实 Attribute |
| [ ] SYN11 | 8.2 弃用 | 双引号/heredoc 中 `${var}`、`${expr}` | 按含义改 `$var`、`{$var}` 或 `{${expr}}`；不混同数组大括号下标 |
| [ ] SYN12 | 7.0 变化 | `$$x['k']`、`$o->$p['k']`、动态方法调用 | 间接访问按从左到右解析；用明确 `{...}` 保留预期变量/成员选择 |
| [ ] SYN13 | 7.0 变化 | 空 `list()`、用 list 解包字符串、list 同数组追加 | 空 list 不再允许；字符串显式分解；核对赋值顺序 |
| [ ] SYN14 | 7.3 变化 | heredoc/nowdoc 结束标签 | 缩进中的同名标记也可能结束字串；避免正文与结束标记冲突 |
| [ ] SYN15 | 7.3 警告 | switch 内 `continue` | 结束 switch 用 break；继续外层循环才用正确层数的 continue |
| [ ] SYN16 | 7.0 变化 | 非法八进制如 `0128`、字面 `\u{` | 前者解析失败；双引号中非 Unicode 转义用途需正确转义反斜杠 |
| [ ] SYN17 | 8.0 变化 | 名称空间分隔符周围空格、源码中的非法控制字节 | 修为合法名称和编码，检查生成的 PHP 源码 |
| [ ] SYN18 | 跨版本约束 | 新加 `??`、`?->`、联合类型、match、enum、readonly 等 | 分别核对语法引入版本；仍支持旧 PHP 时不能放入其会解析的共享文件 |
| [ ] SYN19 | 5.4 移除 | `break $n`、`continue $n`、计算式层数、break/continue 0 | 使用合法常量层数并核对实际嵌套关系 |
| [ ] SYN20 | 7.0 编译期检查 | 循环/switch 外的 break、continue，含 return 后等不可达代码 | 编译阶段也会报错；按真实控制流移除无效语句或修正所在结构，不能因运行不到就忽略；与 SYN15 的 switch 层级问题区分 |

依据：[PHP 5.4 发布记录](https://www.php.net/releases/5_4_0.php)、[7.0 迁移](https://www.php.net/manual/en/migration70.incompatible.php)、[7.3 迁移](https://www.php.net/manual/en/migration73.incompatible.php)、[7.4 弃用](https://www.php.net/manual/en/migration74.deprecated.php)、[运算符优先级](https://www.php.net/manual/en/language.operators.precedence.php)、[8.2 弃用](https://www.php.net/manual/en/migration82.deprecated.php)。

## 2. 已移除的函数、别名、参数形式与扩展

| 状态 / 编号 | 移除版本 | 搜索对象 | 替代 / 核对要点 |
| --- | --- | --- | --- |
| [ ] REM01 | 5.4 | session_register / session_unregister / session_is_registered | 显式操作 `$_SESSION`，核对键存在与 session 状态 |
| [ ] REM02 | 7.0 | `mysql_*` | mysqli 或 PDO；不是给 mysql_* 加空壳函数 |
| [ ] REM03 | 7.0 | ereg / eregi / ereg_replace / eregi_replace / split / spliti / sql_regcase | PCRE 对应接口；重新核对表达式语法、捕获和返回值 |
| [ ] REM04 | 7.0 | ext/mssql、sybase_ct | 选用目标版本可用的相应驱动，核对 API 契约 |
| [ ] REM05 | 7.0 | `preg_replace` 的 `/e` 修饰符 | `preg_replace_callback`；不使用 eval 仿真 |
| [ ] REM06 | 7.0 | call_user_method / call_user_method_array | 明确对象 callable；call_user_func / call_user_func_array 或直接调用 |
| [ ] REM07 | 7.0 | set_magic_quotes_runtime / magic_quotes_runtime | 删除对已不存在机制的依赖；不要重复转义 |
| [ ] REM08 | 7.0 | set_socket_blocking | stream_set_blocking |
| [ ] REM09 | 7.0 | datefmt_set_timezone_id / IntlDateFormatter::setTimeZoneID | datefmt_set_timezone / setTimeZone |
| [ ] REM10 | 7.0 / 7.2 | mcrypt_generic_end、mcrypt_ecb/cbc/cfb/ofb；内置 mcrypt 扩展 | 旧别名 7.0 移除；扩展 7.2 移至 PECL。迁移 OpenSSL/Sodium 时验证算法、格式、IV 和历史兼容，不能只改函数名 |
| [ ] REM11 | 8.0 | each | 通常用 foreach；若原循环操作内部指针或一次消耗多个元素，核对 key/current/next 的推进次数、初始指针、键 0 及末尾 false，不能仅按语法替换 |
| [ ] REM12 | 8.0 | create_function | 闭包；检查 use 捕获、作用域与回调标识 |
| [ ] REM13 | 8.0 | __autoload | spl_autoload_register；清除同一被加载文件中的旧函数定义，不能仅用 function_exists 或版本条件包住它来躲过编译错误；加载入口后验证真实类自动加载及重复注册行为 |
| [ ] REM14 | 8.0 | get_magic_quotes_gpc / get_magic_quotes_runtime | 旧机制 5.4 已删除；按实际输入处理链移除判断 |
| [ ] REM15 | 8.0 | `implode($parts, $separator)` / join 同形式 | 改 `implode($separator, $parts)`；单数组参数形式仍支持 |
| [ ] REM16 | 8.0 | parse_str / mb_parse_str 单参数形式 | 必须提供结果数组参数，显式读取结果 |
| [ ] REM17 | 8.0 | money_format | intl NumberFormatter；核对 locale 与格式 |
| [ ] REM18 | 8.0 | fgetss / SplFileObject::fgetss | 按目标读入再处理文本；strip_tags 不是 HTML 安全过滤器 |
| [ ] REM19 | 8.0 | hebrevc / convert_cyr_string | 分别评估 Unicode RTL、已知源编码的转换接口 |
| [ ] REM20 | 8.0 | ezmlm_hash / restore_include_path / is_real | 前者检查必要性；后两者用 ini_restore('include_path') / is_float |
| [ ] REM21 | 8.0 | image2wbmp / png2wbmp / jpeg2wbmp | 核对实际图像格式及 GD 替代调用 |
| [ ] REM22 | 8.0 | gmp_random | gmp_random_range / gmp_random_bits |
| [ ] REM23 | 8.0 | imap_header；imap_headerinfo 的 default_host 参数 | 使用 imap_headerinfo 并更新参数列表 |
| [ ] REM24 | 8.0 | ldap_sort / ldap_control_paged_result / ldap_control_paged_result_response | 调整排序及 LDAP control 的正式接口 |
| [ ] REM25 | 8.0 | mbregex 旧别名：mbereg、mberegi、mbereg_replace、mberegi_replace、mbsplit、mbereg_match、mbereg_search*、mbregex_encoding | 对应 mb_ereg* / mb_eregi* / mb_split / mb_regex_encoding；逐项核对完整别名表 |
| [ ] REM26 | 8.0 | FILTER_SANITIZE_MAGIC_QUOTES / string.strip_tags stream filter | 核对原转换意图；不把通用转义当参数化或输出编码 |
| [ ] REM27 | 8.0 | FILTER_FLAG_SCHEME_REQUIRED / FILTER_FLAG_HOST_REQUIRED；INPUT_REQUEST / INPUT_SESSION | 删除失效常量用法，按目标 API 明确来源 |
| [ ] REM28 | 8.0 | INTL_IDNA_VARIANT_2003 / Normalizer::NONE | 使用支持的 IDNA/Unicode 选项，验证输出 |
| [ ] REM29 | 8.0 | Reflection 的 export 方法；SplFixedArray 的旧 Iterator 方法 | Reflection 对象转字符串；SplFixedArray 获取 getIterator 后迭代 |
| [ ] REM30 | 8.0 | define 第三个参数 true、大小写不敏感常量 | 使用大小写一致的正式常量名 |
| [ ] REM31 | 8.0 | ext/xmlrpc 随 PHP 捆绑的假设 | 8.0 起不再捆绑；核对替代实现或可用独立扩展，区别“移出捆绑”和“语言禁止” |
| [ ] REM32 | 7.4 | ext/wddx、ext/recode、ext/interbase 的内置发行依赖 | 7.4 移出 PHP 发行；核对目标扩展或替代 API |
| [ ] REM33 | 8.0 | mb_ereg_replace 的 e 修饰符 | 使用 mb_ereg_replace_callback |
| [ ] REM34 | 8.0 | mb_strrpos 第三参作为 encoding；mb_decode_numericentity 的 is_hex 参数 | 明确 offset 与第四参 encoding；删除已移除参数 |
| [ ] REM35 | 7.0 | GD imagepsbbox / imagepsencodefont / imagepsextendfont / imagepsfreefont / imagepsloadfont / imagepsslantfont / imagepstext | Type1 字体支持移除，评估 TrueType 对应接口 |
| [ ] REM36 | 8.0 | read_exif_data | 使用 exif_read_data |
| [ ] REM37 | 8.0 | 从未实现的旧 DOM 类/方法：DOMNameList、DOMConfiguration、DOMNamedNodeMap::setNamedItem 等 | 按官方完整移除表核对；不能假定占位 API 仍存在 |

函数状态可交叉查：[each](https://www.php.net/manual/en/function.each.php)、[create_function](https://www.php.net/manual/en/function.create-function.php)、[preg_replace](https://www.php.net/manual/en/function.preg-replace.php)、[implode](https://www.php.net/manual/en/function.implode.php)、[parse_str](https://www.php.net/manual/en/function.parse-str.php)。扩展与其他别名见[7.0 移除扩展](https://www.php.net/manual/en/migration70.removed-exts-sapis.php)、[7.2 mcrypt 变化](https://www.php.net/manual/en/migration72.other-changes.php)、[7.4 移除扩展](https://www.php.net/manual/en/migration74.removed-extensions.php)、[8.0 扩展变化](https://www.php.net/manual/en/migration80.incompatible.php)。

## 3. PHP 8.2 仍可遇到的弃用项

| 状态 / 编号 | 开始弃用 | 搜索 / 检查对象 | 处理要点 |
| --- | --- | --- | --- |
| [ ] DEP01 | 8.1 | 内置非 nullable 标量参数接收 null：trim、strlen、str_pad、strpos/stripos、strtolower/strtoupper、htmlspecialchars/htmlentities、substr/substr_replace、preg_replace/preg_replace_callback、json_decode；含 array_filter/array_map 等间接调用及 DateTime/DateTimeImmutable 构造器的 datetime 参数 | 先确认空值契约，再在需要字符串的最小边界处理；自定义回调仍保留原始输入。长度过滤回调须保留原先合法的 0/'0'，不能直接换成无回调 array_filter 或 empty；非法结构仍拒绝，不普遍强转。覆盖两种 preg 替换的 subject 及 substr_replace 的 replacement，调试格式化修复不应改变实际绑定值。日期文本原意为当前时间时显式用 'now' 并保留时区，不能把合法可空的 timezone 参数也一并改写。缺失请求字段、CLI 无 HTTP 环境及嵌套输入中的 null 也须覆盖；只在内置字符串调用边界兼容，不顺带改变上层返回值。JSON 解码另区分未取得值、空串、合法 JSON null、损坏 JSON 及合法标量，保留缺省回退顺序和返回类型；不靠真值过滤丢弃 false、0 或 '0'。未使用 JSON_THROW_ON_ERROR（7.3 起）时紧邻解码检查 json_last_error；若本次跳过解码，不得拿前一次 JSON 调用的错误状态判断本次结果。 |
| [ ] DEP02 | 8.1 | int 形参/返回类型、数组键及位运算/取模等整数运算中，有精度损失的隐式 float / 浮点数值字符串转换；含 `/` 的结果继续参加整数运算 | 明确原算法的舍入方向、符号和范围，不静默截断。只有已确认整数输入、整除意图及范围后才评估 intdiv（PHP 7.0 起）；不能无条件用 floor 或整数强转替换所有除法。保留实际需要的浮点计算，并比较零、正负非整除值和整数边界的旧/新结果；按原契约核对后续位运算或编码输出 |
| [ ] DEP03 | 8.1 | ctype_* 非字符串参数 | 区分字符文本与 ASCII 码点；显式字符串或 chr |
| [ ] DEP04 | 8.1 | `$a=false; $a[]=...` | 初始化正确数组；null/未定义值自动形成数组在 8.2 并非同一弃用项 |
| [ ] DEP05 | 8.1 | 内部接口/内部类重写方法缺兼容返回类型，含 trait 提供及子类继承的实现 | 优先补真实签名；ReturnTypeWillChange 仅作过渡，不能修复真实 LSP 不兼容。须保留旧 PHP 时将标记独立成行，并在各支持版本实际加载 ArrayAccess/Iterator/IteratorAggregate/Countable/JsonSerializable/SessionHandlerInterface 等实现类，不只做语法检查。按各方法分别核对契约：如 offsetExists 返回 bool、offsetGet 返回 mixed、offsetSet/offsetUnset 返回 void、getIterator 返回 Traversable、count 返回 int；此处是目标契约对照，不能把旧 PHP 不支持的类型直接写入共享声明。补丁须保留引用返回及方法修饰符，trait 中的修复通过实际使用类验证。回归须执行读写、删除、存在判断、计数、迭代、JSON 输出及适用的会话方法返回值，并保留原返回值、缺失键与异常行为 |
| [ ] DEP06 | 8.1 | Serializable 未同时实现 __serialize / __unserialize | 补现代接口并核对序列化字段、可见性及恢复状态；验证历史载荷读取和新格式往返。新写出格式可能改变，不能把“旧数据能读”当作双向格式完全一致 |
| [ ] DEP07 | 8.1 | 直接访问 trait 的静态方法/属性 | 通过使用 trait 的类访问 |
| [ ] DEP08 | 8.1 | 非数组 __sleep 返回值；void 函数返回引用 | 修正方法契约 |
| [ ] DEP09 | 8.1 | strftime / gmstrftime / strptime / date_sunrise / date_sunset | 按 locale 需求选 Date/Intl 对应接口；纯 UTC 格式可评估 gmdate，但格式占位符不能照抄，须核对时区、字段、边界及超过 24 小时的表示。UTC 分秒格式 `%M:%S` 对应 `i:s`，不能照搬为 `M:S`；累计时长的总小时不能用会按日回绕的日期小时字段替代。至少比较 0、59、60、3599、3600、跨日及负值在原契约下的结果；日出日落用 date_sun_info |
| [ ] DEP10 | 8.1 | FILTER_SANITIZE_STRING / FILTER_SANITIZE_STRIPPED / filter.default | 按原意拆分验证与转换，不能统一替换为 htmlspecialchars |
| [ ] DEP11 | 8.1 | mhash / mhash_keygen_s2k / mhash_count / mhash_get_block_size / mhash_get_hash_name | 迁移 hash_*，核对算法与格式 |
| [ ] DEP12 | 8.1 | imagepolygon / imageopenpolygon / imagefilledpolygon 的旧 num_points 参数 | 使用新的参数形式 |
| [ ] DEP13 | 8.1 | key/current/next/prev/reset/end 传普通 object | 使用正式迭代接口或按意图取对象变量，不假定对象等于数组 |
| [ ] DEP14 | 8.1 | mb_check_encoding 无参数；IntlCalendar::roll 的 bool 参数 | 显式传数据；roll 使用有意义的整数方向 |
| [ ] DEP15 | 8.1 | mysqli_driver::$driver_version；get_client_info 的连接参数；mysqli::init | 按手册用正式版本字段/无参接口/构造函数 |
| [ ] DEP16 | 8.1 | PDO::FETCH_SERIALIZE；PgSQL 隐式连接；odbc_result_all | 显式获取/序列化、显式连接及正式结果遍历 |
| [ ] DEP17 | 8.1 | SoapClient ssl_method；IMAP NIL；oci8.old_oci_close_semantics | 按扩展规则改 stream context/常量/连接管理 |
| [ ] DEP18 | 8.0 | usort/uasort/uksort 等比较器返回 bool | 返回负整数、0、正整数，例如适用时使用 `<=>` |
| [ ] DEP19 | 8.0 | zip_open / zip_read / zip_entry_* 过程 API | 使用 ZipArchive |
| [ ] DEP20 | 8.0 | libxml_disable_entity_loader | 核对实际 libxml 版本、外部加载器和 LIBXML_NOENT/DTDLOAD/DTDVALID 等解析选项；只有等效保护成立才移除或按版本隔离旧调用，必要时正确恢复旧加载状态，并用隔离外部实体用例验证。不能只加 function_exists（8.2 中函数仍可能存在），也不能使用 8.4 才有的 LIBXML_NO_XXE |
| [ ] DEP21 | 8.0 | ReflectionParameter::getClass / isArray / isCallable | 使用 getType 及 ReflectionType 子类型判断 |
| [ ] DEP22 | 8.2 | 未声明的普通对象成员 | 优先按真实读写契约声明属性，保留可见性、初始值与继承行为；stdClass、魔术访问及 AllowDynamicProperties 的例外分别核对，不给所有类统一加标记 |
| [ ] DEP23 | 8.2 | `self::m`、`parent::m`、`static::m` 等相对 callable 形式 | 改明确类/对象 callable，保留作用域语义 |
| [ ] DEP24 | 8.2 | utf8_encode / utf8_decode | 原本仅 ISO-8859-1↔UTF-8；已知源编码后改适当转换 |
| [ ] DEP25 | 8.2 | mbstring 使用 QPrint/Base64/Uuencode/HTML-ENTITIES 伪文本编码 | 使用相应编码/实体函数，不当字符集转换 |
| [ ] DEP26 | 8.2 | SplFileInfo::_bad_state_ex | 不依赖该内部方法 |
| [ ] DEP27 | 7.4 | allow_url_include；进制转换函数中的非法字符 | 删除废弃配置依赖；校验 base_convert/bindec/octdec/hexdec 输入 |

`${...}` 插值弃用已列 SYN11，不重复计算。依据：[8.0 弃用](https://www.php.net/manual/en/migration80.deprecated.php)、[8.1 弃用](https://www.php.net/manual/en/migration81.deprecated.php)、[8.2 弃用](https://www.php.net/manual/en/migration82.deprecated.php)、[utf8_encode](https://www.php.net/manual/en/function.utf8-encode.php)、[DateTime 构造器](https://www.php.net/manual/en/datetime.construct.php)、[intdiv](https://www.php.net/manual/en/function.intdiv.php)、[ReturnTypeWillChange](https://www.php.net/manual/en/class.returntypewillchange.php)、[SessionHandlerInterface](https://www.php.net/manual/en/class.sessionhandlerinterface.php)、[JsonSerializable](https://www.php.net/manual/en/class.jsonserializable.php)。

## 4. 类型、数值、字符串和数组行为

| 状态 / 编号 | 变化版本 | 搜索 / 检查对象 | 核对要点 |
| --- | --- | --- | --- |
| [ ] TYP01 | 7.2 警告 → 8.0 TypeError | count / sizeof 非 array、非 Countable | 先确认容器；不一律 `(array)` 掩盖错误 |
| [ ] TYP02 | 7.4 弃用 → 8.0 不支持 | array_key_exists 的第二参为 object | 按语义选 property_exists 或 isset；两者对 null 不同 |
| [ ] TYP03 | 8.0 变化 | 数字与非数字字符串的 `== != < > <= >=`、switch，以及含这些比较的 `&&` / `\|\|` 短路条件 | 例如 `0 == ''` 从 true 变 false；规范类型后决定比较，不能批量改 ===。复核真假变化后新进入的分支，尤其是后续切片、数值运算与缺省处理；不能只修单个比较而漏掉同一条件链的新异常 |
| [ ] TYP04 | 7.0 变化 | '0x10' 等十六进制字符串当数字 | is_numeric/算术不再按旧数字语义；显式按进制解析 |
| [ ] TYP05 | 7.1 变化 | intval / `(int)` / settype 处理科学计数法 | 核对 '1e3' 等转换结果与范围 |
| [ ] TYP06 | 8.0 变化 | is_numeric、算术、类型转换中的空白和非数字串；substr/解码结果参与数值运算 | 尾随空白可以成为合法数值串；非法算术可能 TypeError。固定宽度文本先按原契约检查长度、字符格式和完整性，再转换与运算；覆盖空串、截断、部分数字、非数字及数值边界。不能先把非法输入强转 0 充当合法缺省值，也不能用 is_numeric 代替更严格的原格式约束 |
| [ ] TYP07 | 8.0 变化 | 数组/资源/普通对象参与算术或位运算 | 核对 TypeError；数组 `+` 数组仍是并集操作 |
| [ ] TYP08 | 7.0 / 8.0 | 除零、模零、负移位 | 8.2 `/`、`%` 除零抛 DivisionByZeroError；负移位抛 ArithmeticError |
| [ ] TYP09 | 8.0 变化 | 字符串查找函数 needle 为整数 | 不再按 ASCII 码点；显式 chr 或字符串。同时检查空 needle 与偏移边界 |
| [ ] TYP10 | 8.0 变化 | substr / mb_substr / iconv_substr / grapheme_substr 越界 | 一些原 false 返回变空串；核对严格比较与返回类型 |
| [ ] TYP11 | 8.0 变化 | substr/substr_count/substr_compare/iconv_substr 的 null 长度 | 部分接口按未传长度解释；逐函数核对，不假设都是长度 0 |
| [ ] TYP12 | 8.0 变化 | array_splice 的 null 长度 | 按省略参数处理到末尾，避免误删数量 |
| [ ] TYP13 | 8.0 变化 | parse_url 的 query / fragment | 区分不存在 null 与存在但空 '' |
| [ ] TYP14 | 8.0 变化 | 稳定排序、相等元素原始次序 | 等值元素维持原先顺序；比较器仍需正确三态返回 |
| [ ] TYP15 | 7.2 变化 | array↔object 强转的数字键 | 核对数字键可访问性与键类型 |
| [ ] TYP16 | 7.0 变化 | foreach 与 current/key/next；遍历中增删 | 不再改变内部指针；按值/按引用遍历变动规则不同 |
| [ ] TYP17 | 7.0 / 8.0 | 数组引用创建顺序、负数字键后的隐式键 | 核对元素顺序及后续自动编号 |
| [ ] TYP18 | 8.0 变化 | float 转 string；默认 locale | 浮点转串不再受 locale 影响；locale 不再靠环境隐式继承 |
| [ ] TYP19 | 7.2 变化 | number_format 的负零 | 原 '-0' 输出可能变 '0' |
| [ ] TYP20 | 8.1 变化 | htmlspecialchars / htmlentities 默认 flags | 默认为 ENT_QUOTES + ENT_SUBSTITUTE + ENT_HTML401；需要稳定输出时明确 flags 和编码 |
| [ ] TYP21 | 8.2 变化 | strtolower/strtoupper/stripos/stristr 等、array_change_key_case、SORT_FLAG_CASE | 大小写处理采用 ASCII 规则；非 ASCII 需求核对 mbstring |
| [ ] TYP22 | 8.2 变化 | `str_split('')` | 返回 []，不再是含一个空字符串的数组 |
| [ ] TYP23 | 8.2 变化 | ksort / krsort 的 SORT_REGULAR | 数字字符串按 PHP 8 标准比较规则排序 |
| [ ] TYP24 | 8.2 变化 | var_export 对象类名 | 导出类名增加前导反斜杠；核对依赖导出文本的代码 |
| [ ] TYP25 | 5.6 变化 | json_decode 的 TRUE / False / NULL 等非规范字面量 | 必须使用规范小写；检查解码错误，不把 null 当自动成功 |
| [ ] TYP26 | 7.3 变化 | 自定义 ArrayAccess 的整数型字符串字面键 | 不再自动转 int 后传给 offsetGet；核对自定义键处理 |
| [ ] TYP27 | 7.1 变化 | srand/mt_srand、rand/mt_rand、shuffle/array_rand | 算法与别名变化；依赖固定种子序列时做结果对照 |
| [ ] TYP28 | 7.2 警告 → 8.0 TypeError | get_class(null) 等依赖 null 推断类的调用 | 传实际对象；类内省略参数与显式 null 不是同一契约 |
| [ ] TYP29 | 8.2 变化 | strcmp/strcasecmp/strncmp/strncasecmp/substr_compare 返回值 | 只依赖小于/等于/大于 0；不能依赖长度差等具体幅值 |

函数依据：[isset](https://www.php.net/manual/en/function.isset.php)、[empty](https://www.php.net/manual/en/function.empty.php)、[count](https://www.php.net/manual/en/function.count.php)、[array_key_exists](https://www.php.net/manual/en/function.array-key-exists.php)、[比较规则](https://www.php.net/manual/en/language.operators.comparison.php)、[strpos](https://www.php.net/manual/en/function.strpos.php)、[substr](https://www.php.net/manual/en/function.substr.php)、[array_splice](https://www.php.net/manual/en/function.array-splice.php)、[parse_url](https://www.php.net/manual/en/function.parse-url.php)、[htmlspecialchars](https://www.php.net/manual/en/function.htmlspecialchars.php)。其余按[5.6](https://www.php.net/manual/en/migration56.incompatible.php)、[7.1](https://www.php.net/manual/en/migration71.incompatible.php)、[7.2](https://www.php.net/manual/en/migration72.incompatible.php)、[7.3](https://www.php.net/manual/en/migration73.incompatible.php)、[8.2](https://www.php.net/manual/en/migration82.incompatible.php)对应章节核对。

## 5. 类、方法、继承、回调与反射

| 状态 / 编号 | 变化版本 | 搜索 / 检查对象 | 核对要点 |
| --- | --- | --- | --- |
| [ ] OOP01 | 8.0 变化 | 与类同名的 PHP 4 构造方法 | 不再自动当构造器；改 __construct，核对父构造调用 |
| [ ] OOP02 | 8.0 不支持 | 非 static 方法用 `C::m()` 调用 | 需要实例则使用对象；不无差别添加 static |
| [ ] OOP03 | 8.0 严格化 | 父类/接口实现的参数、返回值、可见性、static | LSP 不兼容可能从警告升级为致命错误；核对所有实现 |
| [ ] OOP04 | 8.0 严格化 | 魔术方法签名、trait 抽象方法签名 | 与目标规则兼容，实际加载类验证 |
| [ ] OOP05 | 8.0 编译期致命 | 无父类时用 parent，含 parent::__construct；trait 别名未指明冲突来源 | 先核对真实继承关系；确需父类时恢复正确继承契约，独立类遗留的无效父调用则在确认自身初始化完整后移除，不能为消错随意增加父类。修后同时验证编译、类加载、构造与原有初始化结果；trait 别名须写明正确来源 trait |
| [ ] OOP06 | 8.0 / 8.1 | 必选参数在可选参数之后 | 8.0 开始弃用；8.1 命名调用也不能靠默认值跳过。在确认位置/命名调用及 nullable 类型例外后，删除确已失效的默认值；保留参数顺序、`&` 引用标记及输出参数契约，并核对失败分支的输出值。不擅自重排参数，也不批量给后续必选参数补空值 |
| [ ] OOP07 | 7.1 变化 | 用户函数缺少必选实参 | ArgumentCountError；检查回调、动态分发和旧 handler |
| [ ] OOP08 | 8.0 变化 | call_user_func_array 的字符串键 | 作为命名参数；未知名/顺序混合可能错误，只有纯位置契约才用 array_values |
| [ ] OOP09 | 8.0 变化 | 动态分发/反射 invokeArgs/参数解包 | 核对命名参数与引用规则；公共参数名不可随意改 |
| [ ] OOP10 | 7.1 限制 | 动态调用 extract、compact、func_get_*、get_defined_vars 等作用域函数 | 改明确直接调用，核对特殊限制 |
| [ ] OOP11 | 8.0 变化 | array_filter / array_reduce 等回调声明引用形参 | 核对调用 API 是否承诺按引用传参，否则会警告 |
| [ ] OOP12 | 8.0 变化 | Closure::bind/解绑 $this | 使用 $this 的闭包及方法转成的闭包不能按旧方式解绑 |
| [ ] OOP13 | 8.1 变化 | 子类继承但未重写方法里的 static 局部变量 | 与父方法共享状态，不再独立计数 |
| [ ] OOP14 | 8.0 变化 | final private 方法、构造中 exit/异常、未捕获异常后的析构 | 核对诊断和析构实际执行；构造器为部分规则的例外 |
| [ ] OOP15 | 8.0 变化 | ReflectionType::isBuiltin 等单类型假设 | 使用 ReflectionNamedType/UnionType/IntersectionType 分别处理 |
| [ ] OOP16 | 8.0 变化 | 匿名类名、反射/堆栈参数引用 | 不依赖匿名类名格式，不能通过 backtrace 引用改实参 |
| [ ] OOP17 | 8.2 变化 | 继承 DateTime::createFromImmutable / DateTimeImmutable::createFromMutable | 暂定返回类型变 static，核对重写签名 |
| [ ] OOP18 | 8.2 变化 | 重写 SplFileObject getCsvControl/fflush/ftell/fgetc/fpassthru；SplFileInfo::_bad_state_ex | 方法签名更严格；核对参数数目与类型 |
| [ ] OOP19 | 8.2 变化 | SplFileObject::hasChildren / getChildren | 暂定返回类型分别为 false / null；核对继承方法 |
| [ ] OOP20 | 8.2 变化 | iterable 联合类型的反射结果 | 编译时作为 array + Traversable 的类型别名；单独 iterable 的反射仍保留命名类型 |

依据：[call_user_func_array](https://www.php.net/manual/en/function.call-user-func-array.php)、[8.0 类与调用变化](https://www.php.net/manual/en/migration80.incompatible.php)、[8.1 继承变化](https://www.php.net/manual/en/migration81.incompatible.php)、[8.2 方法变化](https://www.php.net/manual/en/migration82.incompatible.php)。动态属性及内部方法返回类型见 DEP05/DEP22，不重复列。

## 6. 错误、异常和非法值处理

| 状态 / 编号 | 变化版本 | 检查对象 | 核对要点 |
| --- | --- | --- | --- |
| [ ] ERR01 | 7.0 变化 | catch(Exception)、set_exception_handler 的 Exception 类型声明 | Error 也实现 Throwable；需要同时处理时使用合适 Throwable 边界 |
| [ ] ERR02 | 7.0 / 8.0 | 内置函数非法类型、个数或范围 | 大量旧 warning/false 改 TypeError、ArgumentCountError、ValueError；逐函数查手册 |
| [ ] ERR03 | 8.0 警告提升 | 未定义变量/属性/数组键、非数组下标、非对象属性、数组转串；直接对函数返回值取下标 | Notice 可能变 Warning；在首次读取前初始化并修正来源。允许返回 false/null 的 API 先核对成功/失败契约和容器类型，再读下标或嵌套字段。按含义选 isset、array_key_exists、empty 或明确缺省值：null、false、0、'0' 不可混同；仅可选值按原契约缺省，不能把必要数据失败伪装成成功，也不用 @ 掩盖 |
| [ ] ERR04 | 8.0 致命化 | 向 null/false/空字符串等非对象写属性；解码结果、构造中断后的异常分支 | 不再隐式形成 stdClass；先校验 JSON 解码结果、结构及必要字段，再创建正确对象。catch/finally 内仅操作确已初始化的实例，不能用任意强转掩盖错误或引发第二次异常。实例存在判断还须符合预期类/接口；仅 is_object 不保证后续成员或方法可用。分别验证实例创建前失败和创建后失败，保留根异常，不为完成诊断而创建带额外写入的新对象。 |
| [ ] ERR05 | 8.0 严格化 | 标量写数组下标、非法数组键/字符串偏移、非法解包 | 修容器/键/元素类型，不能仅补默认键 |
| [ ] ERR06 | 8.0 变化 | @ 和自定义错误处理器的 error_reporting 判断 | 不能再假设 @ 下恒为 0；按错误位掩码检查 |
| [ ] ERR07 | 8.0 移除参数 | set_error_handler 回调第五个 errcontext | 去除必选第五参数，核对兼容签名 |
| [ ] ERR08 | 8.0 变化 | assert 字符串表达式、断言异常默认值 | 字符串不再执行；必须写真实表达式；断言不能替代运行时必要检查 |
| [ ] ERR09 | 7.3 / 8.0 | compact 引用未定义变量 | 7.3 Notice、8.0 Warning；先明确定义变量 |
| [ ] ERR10 | 8.0 变化 | `disable_functions` / function_exists / Reflection | 被禁用函数视同不存在；核对实际 SAPI 的生效配置，不因 function_exists 返回 false 就误判整个 PHP API 已移除。INI 指令名为 disable_functions，不是 disabled_functions |
| [ ] ERR11 | 8.0 默认变化 | error_reporting、display_startup_errors | 默认报告范围/启动错误展示变化；显式核对 ini，不把旧静默当兼容 |
| [ ] ERR12 | 8.2 变化 | random_bytes / random_int 的 CSPRNG 失败 | 异常细分为 Random\RandomException；仍是 Exception 子类，核对精确类型匹配 |

依据：[PHP 7 错误体系](https://www.php.net/manual/en/migration70.incompatible.php)、[Throwable](https://www.php.net/manual/en/class.throwable.php)、[assert](https://www.php.net/manual/en/function.assert.php)、[compact](https://www.php.net/manual/en/function.compact.php)、[PHP 8 错误变化](https://www.php.net/manual/en/migration80.incompatible.php)。DEP01 在弱转换模式下涉及弃用，strict_types 情况可能直接 TypeError，需分别检查。

## 7. 内置扩展、句柄、数据库与协议默认值

这里只列由 PHP 发行版本/API 改变引起的项目；任意第三方扩展漏装、服务不可达不归入本表。

| 状态 / 编号 | 变化版本 | 检查对象 | 核对要点 |
| --- | --- | --- | --- |
| [ ] EXT01 | 5.6 | GMP resource → GMP object | 检查 is_resource、强转 int、资源编号依赖 |
| [ ] EXT02 | 8.0 | CurlHandle/CurlMultiHandle/CurlShareHandle、GdImage、OpenSSL* 对象 | 成功句柄不再是 resource；按具体接口 false/异常契约判断，并检查强转、动态属性名、数组键等对旧资源标识的依赖，不把句柄对象当成字符串名称 |
| [ ] EXT03 | 8.0 | Socket/AddressInfo、Shmop、Sysv*、XMLParser/XMLWriter、InflateContext/DeflateContext 等 | 同查资源判断/清理接口/强制转换，不只搜 curl |
| [ ] EXT04 | 8.1 | Fileinfo、FTP、IMAP、LDAP、PgSQL、PSpell resource → object | 更新句柄判断及类型约束，按实际类核对 |
| [ ] EXT05 | 8.0 | PDO 默认错误模式 | 默认变 ERRMODE_EXCEPTION；检查异常边界，不再仅等 false |
| [ ] EXT06 | 8.1 | mysqli 默认报告模式 | 默认为 ERROR + STRICT，可能抛 mysqli_sql_exception；不要用统一关闭报告代替修复 |
| [ ] EXT07 | 8.1 | PDO MySQL 模拟预处理 / SQLite 的数字结果；ATTR_STRINGIFY_FETCHES | 部分原字符串变原生 int/float；bool stringify 为 '0'/'1'；核对严格类型判断 |
| [ ] EXT08 | 8.1 | PDO bindColumn 的 PARAM_LOB | 特定选项下统一绑定 stream，不能假定总是 string |
| [ ] EXT09 | 8.1 | mysqli 元数据 max_length、store_result 选项、connect 成功返回、mysqli_stmt::execute 重写 | max_length 为 0、旧选项失效、成功为 true；更新继承参数 |
| [ ] EXT10 | 5.6 | SSL/TLS stream 默认 peer / name 校验 | 正确设置 CA 与主机名；不以关闭校验作为迁移方案 |
| [ ] EXT11 | 8.0 | HTTP stream wrapper 默认 HTTP/1.1 | 核对协议交互；需旧行为时明确 protocol_version |
| [ ] EXT12 | 5.6 / 7.0 | cURL 的 `@file` 上传形式、CURLOPT_SAFE_UPLOAD | 使用 CURLFile，不能依赖关闭旧安全上传模式 |
| [ ] EXT13 | 7.3 | PCRE → PCRE2 | 实际运行正则并检查 preg_last_error；尤其转义、重复组、无效模式 |
| [ ] EXT14 | 8.0 | crypt 的默认 salt / 未识别 salt / rounds；password_hash 的 salt 选项 | 默认 salt 用法不再支持、错误返回规则变化；手工 salt 选项失效 |
| [ ] EXT15 | 8.1 | OpenSSL EC 私钥导出、PKCS7/CMS 默认密码算法 | 私钥格式/默认算法改变；核对互通格式，不假定二进制不变 |
| [ ] EXT16 | 7.4 | password_* 算法常量 | 从整数标识变字符串标识（早期 7.4 补丁另有差异）；不要硬编码整数 |
| [ ] EXT17 | 8.0 | imagecropauto mode=-1；Windows php_gd2.dll | 使用 IMG_CROP_DEFAULT；Windows 库名改 php_gd.dll |
| [ ] EXT18 | 8.2 | ODBC/PDO_ODBC 连接参数转义 | 核对组合连接串与独立用户名/密码的转义结果 |
| [ ] EXT19 | 8.2 | glob / GlobIterator + open_basedir | 全部被限制时 [] 而非 false；部分受限也会警告 |
| [ ] EXT20 | 8.2 | FilesystemIterator 显式 flags | 需要跳过点目录时显式 SKIP_DOTS；不误改默认 flags 的说明 |
| [ ] EXT21 | 8.2 | 日期相对格式中的多重符号 | 不再接受 '+-2' 等数字符号组合 |
| [ ] EXT22 | 8.0 | mktime() / gmmktime() 无参数 | 至少提供 hour；取当前时间戳用 time() |
| [ ] EXT23 | 8.2 | DateTime/DateTimeImmutable::getLastErrors / date_get_last_errors | 无警告/错误时返回 false，不再始终是数组；先判类型再读计数 |
| [ ] EXT24 | 8.2 | dba_open / dba_popen / dba_fetch | open/popen 签名更严格；fetch 使用 key、handle、skip 的标准顺序（旧重载仍接受） |
| [ ] EXT25 | 8.2 | IntlCalendar、Collator、IntlDateFormatter、ResourceBundle 等 Intl 类的序列化 | 一批此前恢复结果不可用的类正式禁止序列化；按官方列表检查 |
| [ ] EXT26 | 8.2 | mysqli 的 libmysql 构建依赖、mysqli_driver::reconnect、mysqli.reconnect | mysqli 只支持 mysqlnd，原 libmysql 专属功能不可继续依赖 |

依据：[5.6 变化](https://www.php.net/manual/en/migration56.incompatible.php)、[8.0 句柄类目录](https://www.php.net/manual/en/migration80.new-classes.php)、[8.1 句柄/驱动变化](https://www.php.net/manual/en/migration81.incompatible.php)、[PDO 错误模式](https://www.php.net/manual/en/pdo.error-handling.php)、[mysqli 配置](https://www.php.net/manual/en/mysqli.configuration.php)、[8.2 变化](https://www.php.net/manual/en/migration82.incompatible.php)。

补充函数依据：[get_class](https://www.php.net/manual/en/function.get-class.php)、[strcasecmp](https://www.php.net/manual/en/function.strcasecmp.php)、[read_exif_data](https://www.php.net/manual/en/function.read-exif-data.php)、[mktime](https://www.php.net/manual/en/function.mktime.php)、[getLastErrors](https://www.php.net/manual/en/datetimeimmutable.getlasterrors.php)、[8.2 其他变化](https://www.php.net/manual/en/migration82.other-changes.php)。

## 8. INI、超全局变量和引擎配置

| 状态 / 编号 | 变化版本 | 搜索对象 | 核对 / 处理 |
| --- | --- | --- | --- |
| [ ] CFG01 | 5.4 移除 | register_globals / register_long_arrays / 旧 HTTP_*_VARS | 显式使用超全局变量，不恢复自动变量注入 |
| [ ] CFG02 | 5.4 移除 | safe_mode 及 safe_mode_* | 去除失效配置依赖；它不再提供权限约束 |
| [ ] CFG03 | 5.4 移除 | magic_quotes_gpc/runtime/sybase | 不再自动转义；核对旧代码是否假定输入已加反斜杠 |
| [ ] CFG04 | 8.1 限制 | `$GLOBALS` 整体引用、整体赋值、整体写操作 | 改明确元素访问/参数；读取整表和写单个元素仍支持 |
| [ ] CFG05 | 8.0 移除 | track_errors / `$php_errormsg` | 使用适当错误处理和 error_get_last，不能照抄旧变量 |
| [ ] CFG06 | 8.0 移除 | mbstring.func_overload / MB_OVERLOAD_* | 不再以配置重载普通字符串函数；显式选择字节/字符接口 |
| [ ] CFG07 | 7.0 FPM 不支持 | dl() 运行时加载扩展 | 使用目标版本正式加载配置；不要误判 CLI 与 FPM 等价 |
| [ ] CFG08 | 8.1 弃用 | auto_detect_line_endings | 显式处理需要支持的换行形式 |
| [ ] CFG09 | 8.0 移除 | assert.quiet_eval / ASSERT_QUIET_EVAL | 去除旧字符串断言机制相关配置 |
| [ ] CFG10 | 5.6 起相关默认值变化 | default_charset、precision / serialize_precision、session/编码旧 ini | 对依赖序列化/浮点文本/编码默认值的代码逐版比对，不直接复制旧 ini 全文 |
| [ ] CFG11 | 7.0 移除 | `$HTTP_RAW_POST_DATA` / always_populate_raw_post_data | 通过 `php://input` 读取原始请求体 |
| [ ] CFG12 | 7.0 移除 | xsl.security_prefs | 使用 XSLTProcessor::setSecurityPrefs 的实例级设置 |
| [ ] CFG13 | 8.2 变化 | 数值型 INI 的畸形值、0 开头值、0b/0o 前缀 | 部分原静默格式会警告；0 开头仍可能是八进制；按目标解析规则核对 |
| [ ] CFG14 | 8.2 变化 | session 已启动/已输出后修改 session.cookie_samesite | 失败并警告，调整配置时机 |
| [ ] CFG15 | 8.2 变化 | 依赖 PHP 生成 Cookie 到期时间原始文本格式 | 日期格式分隔符有变化；不能依赖旧字节串格式做精确匹配 |

依据：[5.4 官方发布](https://www.php.net/releases/5_4_0.php)、[GLOBALS 限制](https://www.php.net/manual/en/migration81.incompatible.php)、[8.0 配置与扩展变化](https://www.php.net/manual/en/migration80.incompatible.php)、[8.1 弃用配置](https://www.php.net/manual/en/migration81.deprecated.php)。

## 9. 格式与新写法：强制规则、建议和非必需升级

| 核对 | 属性 | 标准 |
| --- | --- | --- |
| PHP 标签、下标、三元、关键字、方法签名 | 引擎兼容要求 | 按前表修复，目标解析器和类加载必须通过 |
| 共享文件的最低语法版本 | 兼容要求 | 支持 5.x/7.x 时，不能因目标为 8.2 就无条件使用 8.x 专属语法 |
| 参数/属性/返回类型和 null | 契约要求 | 类型必须与实际调用和继承兼容；不为消告警给一切增加空串、0 或 mixed |
| PSR-12、UTF-8/LF、4 空格、纯 PHP 无结尾 `?>` | 可选择的风格标准 | 不是 PHP 8.2 强制；既有非 UTF-8 文件不为格式统一直接转码 |
| declare(strict_types=1) | 可选择的类型策略 | PHP 8.2 不要求全量启用；启用会改变调用转换/异常，须核对调用者 |
| typed property、联合类型、命名参数、Attribute、enum、readonly | 可选择的新功能 | 不属于“必须改造”清单；不采用也能符合 PHP 8.2 |

风格依据：[PHP-FIG PSR-12](https://www.php-fig.org/psr/psr-12/)。不要将代码格式化或新语法推广当作解决版本不兼容的必要步骤。

## 10. 快速扫描与核对方法

1. **确定范围。** 记录旧/目标精确 PHP 版本及支持区间，包含实际加载的 PHP、inc、phtml、生成 PHP 等文件。核对真实 Web 请求所使用的解释器、SAPI、FPM 实例/池、INI 与已加载扩展；运行中的映像不以磁盘文件或 CLI 的版本输出代替。补装扩展时核对 phpize/php-config、PHP/Zend API、架构及 ZTS/debug 构建匹配，模块文件存在不等于 Web 已加载。分别记录 CLI/Web 的模块与 libxml 版本，区分缺扩展、API 移除及 Composer 平台/条件加载约束，不绕过平台检查制造兼容结论。
2. **语法扫描。** 用目标解释器逐文件 `php -l`，检查退出码和完整输出；不可达代码及模板产出的 PHP 也必须检查。双版本项目按实际自动加载及版本分发关系标注检查范围，不把只为较新 PHP 条件加载的文件误当作旧版本共享源码；保留失败、排除及未验证清单。语法通过不证明函数、类型或运行结果正确，也不能把局部通过视为整个依赖集兼容。
3. **兼容规则扫描。** 使用已安装且支持目标版本的 PHPCompatibility，记录工具版本和 testVersion。规则未命中不代表没有问题。
4. **按表检查动态用法。** 重点检查变量函数、反射、回调数组、继承、资源类型与默认参数；纯文本搜索不能完整发现这些用法。候选命中按 token/语法上下文复核，区分注释、普通字符串、同名方法与真实调用；将排除的生成代码和重复依赖副本单独列出，不能把扫描命中数或排除数直接当作已修数量。按实际自动加载路径加载相关类，确认缺失类是否由更早的编译/签名错误引起，避免修错未被使用的副本。
5. **运行差异核对。** 在 E_ALL 下执行命中的最小调用；比较值、类型、异常、错误级别。保留修改前的可复现失败及修改后的同输入通过证据；旧环境无法运行时明确记为未验证，不以新版本通过推断双版本通过。样本至少有缺失/null/''/'0'/0/false/[]、非预期对象、边界数值与非 ASCII 文本。回调还核对键和零值保留；指针迭代、序列化及编码转换分别核对消费顺序、旧格式读取及输出字节。
6. **生成代码与输出边界。** 修模板源或编译入口，只失效对应的生成 PHP 缓存后重新编译执行；不能只改缓存文件。另查实际运行时的 OPcache 是否启用及时间戳校验策略，按需在对应实例验证修复后的代码确已执行；不能把磁盘源码已改、CLI 缓存操作成功或删除模板缓存等同于 Web opcode 缓存已更新。OPcache 属可选运行组件，不是 PHP 8.2 强制语言规范。转义处兼容空值时保留原过滤器、显式参数及原有输出契约；不为消除弃用而关闭转义。检查变量默认值是否在首次使用前建立。
7. **完整响应与日志。** 同查 HTTP 状态、原始响应体、JSON 可解析性和 HTML/内联 JavaScript 完整性；Warning 插入响应即可破坏客户端解析，HTTP 200 不代表正常。对照 PHP/FPM 与应用日志的新增窗口，检索原始异常文本和调用栈；同时检查 warning/notice/deprecated，以及被 catch 后写入 info/debug 的异常或失败摘要，按请求时间与堆栈关联最早异常，不只查 error 级别；核对自定义错误处理器是否将 E_DEPRECATED/E_WARNING 转成异常，不降低报告级别来通过验收。升级后遇到被静默处理的白屏，可在受影响调用前临时注册 shutdown 回调，用 error_get_last 核对致命错误类别、文件与行号；须判空并按错误类型过滤，不能把任意最后一条警告当作中断原因，也不能以没有记录推断没有异常。诊断限定到必要入口，不把堆栈、凭据或输入输出到页面，完成后移除诊断代码并复核最终差异。
8. **真实调用链回归。** 在实际支持的各 PHP 版本运行原调用入口及依赖方法，覆盖成功、空输入、非法输入和初始化失败路径；修复首个异常后继续执行同一调用链，检查后续调用。对比较或短路变化补测各分支；涉及时间字段时固定时钟验证边界，涉及摘要、编码或历史格式时验证原有效样本仍可读且损坏/截断数据按原契约拒绝。需要外部 I/O 时在实际传输边界使用隔离替身，断言没有真实外部请求，并核对请求构造、响应解码及从响应生成后续参数的完整链路，覆盖成功与拒绝响应；仅替身返回成功不足以证明后续处理通过。涉及签名/摘要的协议，用固定测试密钥和非敏感输入独立核对字段剔除、键顺序、转义与签名字节；随机字段按协议另行验证，不能把动态整包哈希差异当作失败。涉及文件、图像、摘要及二进制格式时，用合成样例做生成/读取往返，分别核对中文、前导零字符串、数值、布尔及空值的类型和表示；只在原契约要求字节稳定时比较字节或哈希，不把动态元数据差异当作失败。区分单函数、类加载、模板渲染与完整请求链证据；未登录重定向或入口可达不代表后续调用已执行。
9. **补丁复核。** 第三方/vendor 兼容补丁记录最小差异、依赖版本、变更前后哈希与验证命令；重装/升级依赖后复查是否被覆盖。回退前核对当前版本，避免覆盖后续修复。只记录 PHP 兼容性证据，不扩展为项目交付或一般服务器排障清单。

只读候选搜索示例（在代码副本运行，不自动替换）：

```sh
# 移除函数候选；根据前表补充实际使用的扩展接口。
rg -n --glob '*.php' --glob '*.inc' --glob '*.phtml' \
  '\b(mysql_[A-Za-z0-9_]+|ereg(i)?|ereg(i)?_replace|split|spliti|sql_regcase|each|create_function|__autoload|get_magic_quotes_gpc|get_magic_quotes_runtime|money_format|fgetss|is_real|session_register|call_user_method(_array)?)\s*\(' .

# 类型、回调、句柄、配置变化候选；命中不等于错误。
rg -n --glob '*.php' --glob '*.inc' --glob '*.phtml' \
  'count\s*\(|str_pad\s*\(|array_filter\s*\(|array_map\s*\(|json_decode\s*\(|libxml_disable_entity_loader\s*\(|call_user_func_array\s*\(|is_resource\s*\(|array_key_exists\s*\(|\$GLOBALS|Serializable|FILTER_SANITIZE_STRING|utf8_encode|utf8_decode|\$HTTP_RAW_POST_DATA' .

# 在已安装对应标准的隔离工具环境运行。
vendor/bin/phpcs -i
vendor/bin/phpcs . --standard=PHPCompatibility --extensions=php,inc,phtml \
  --runtime-set testVersion 8.2

# 若同一源码必须同时支持 7.4 与 8.2：
vendor/bin/phpcs . --standard=PHPCompatibility --extensions=php,inc,phtml \
  --runtime-set testVersion 7.4-8.2
```

补充核对依据：[数值字符串](https://www.php.net/manual/en/language.types.numeric-strings.php)、[disable_functions](https://www.php.net/manual/en/ini.core.php#ini.disable-functions)、[phpize/php-config](https://www.php.net/manual/en/install.pecl.phpize.php)、[OPcache 配置](https://www.php.net/manual/en/opcache.configuration.php)、[opcache_reset](https://www.php.net/manual/en/function.opcache-reset.php)、[array_filter](https://www.php.net/manual/en/function.array-filter.php)、[json_decode](https://www.php.net/manual/en/function.json-decode.php)、[json_last_error](https://www.php.net/manual/en/function.json-last-error.php)、[ReturnTypeWillChange](https://www.php.net/manual/en/class.returntypewillchange.php)、[libxml_disable_entity_loader](https://www.php.net/manual/en/function.libxml-disable-entity-loader.php)、[gmdate](https://www.php.net/manual/en/function.gmdate.php)、[日期格式字段](https://www.php.net/manual/en/datetime.format.php)、[ArrayAccess](https://www.php.net/manual/en/class.arrayaccess.php)、[spl_autoload_register](https://www.php.net/manual/en/function.spl-autoload-register.php)、[error_get_last](https://www.php.net/manual/en/function.error-get-last.php)、[register_shutdown_function](https://www.php.net/manual/en/function.register-shutdown-function.php)。

工具依据：[PHPCompatibility 官方说明](https://github.com/PHPCompatibility/PHPCompatibility)。工具自身覆盖持续完善，安装要求按实际版本核对。`8.2` 用于目标单版本检查，区间应按实际支持范围填写；不默认所有迁移都要保留旧运行时。

**禁止机械替换：** 全局 `== → ===`、全部 `{} → []`、全部 `continue → continue 2`、给所有类加 AllowDynamicProperties、给所有方法加 mixed、把所有 null 强转空串、对所有异常返回成功。它们会引入新的语义差异。

## 11. 官方逐版本补查索引

前表按检查对象汇总；下面按跨越版本补查不兼容、弃用、移除及 changed functions 章节，尤其是实际使用的可选扩展。

| 跨越版本 | 官方入口 |
| --- | --- |
| 早期 5.x→5.4 | [5.4 发布](https://www.php.net/releases/5_4_0.php)；旧手册页不可用时查官方历史文档 |
| 5.4→5.5 | [5.5 发布](https://www.php.net/releases/5_5_0.php)、具体函数的 changelog |
| 5.5→5.6 | [5.6 迁移](https://www.php.net/manual/en/migration56.php) |
| 5.6→7.0 | [7.0 迁移](https://www.php.net/manual/en/migration70.php) |
| 7.0→7.1 | [7.1 迁移](https://www.php.net/manual/en/migration71.php) |
| 7.1→7.2 | [7.2 迁移](https://www.php.net/manual/en/migration72.php) |
| 7.2→7.3 | [7.3 迁移](https://www.php.net/manual/en/migration73.php) |
| 7.3→7.4 | [7.4 迁移](https://www.php.net/manual/en/migration74.php) |
| 7.4→8.0 | [8.0 不兼容](https://www.php.net/manual/en/migration80.incompatible.php)、[8.0 弃用](https://www.php.net/manual/en/migration80.deprecated.php) |
| 8.0→8.1 | [8.1 不兼容](https://www.php.net/manual/en/migration81.incompatible.php)、[8.1 弃用](https://www.php.net/manual/en/migration81.deprecated.php) |
| 8.1→8.2 | [8.2 不兼容](https://www.php.net/manual/en/migration82.incompatible.php)、[8.2 弃用](https://www.php.net/manual/en/migration82.deprecated.php) |

小版本维护规则：具体函数手册通常同时展示 8.3/8.4/8.5 变化；只把目标不高于 8.2 的规则纳入本清单。不要把 `array_key_exists` 的 null key 弃用、某些 assert 配置项弃用等后续版本事项误写成 8.2 要求。5.x 涵盖多个小版本，来源早于 5.4 时应继续核对该精确旧版本至 5.4 的官方历史变化。

执行结果记录格式：`编号｜旧版本｜目标版本｜命中位置/调用｜状态｜修复方式｜最小验证结果`。记录只需围绕 PHP 差异及其验证，不加入业务流程或项目案例。
