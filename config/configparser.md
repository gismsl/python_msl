[toc]

# 官方文档

- python3 https://docs.python.org/3/library/configparser.html
- Python2 https://docs.python.org/2/library/configparser.html

# 快速入门

`文件的格式如下，ini后缀名的文件`

```ini
[DEFAULT]
ServerAliveInterval = 45
Compression = yes
CompressionLevel = 9
ForwardX11 = yes

[bitbucket.org]
User = hg

[topsecret.server.com]
Port = 50022
ForwardX11 = no
```

## 写ini文件

```python
import configparser
config = configparser.ConfigParser()
config['DEFAULT'] = {'ServerAliveInterval': '45',
                      'Compression': 'yes',
                      'CompressionLevel': '9'}
config['bitbucket.org'] = {}
config['bitbucket.org']['User'] = 'hg'
config['topsecret.server.com'] = {}
topsecret = config['topsecret.server.com']
topsecret['Port'] = '50022'     # mutates the parser
topsecret['ForwardX11'] = 'no'  # same here
config['DEFAULT']['ForwardX11'] = 'yes'
with open('example.ini', 'w') as configfile:
   config.write(configfile)
```

## 读ini文件

```python
>>> config = configparser.ConfigParser()
>>> config.sections()
[]
>>> config.read('example.ini')
['example.ini']
>>> config.sections()
['bitbucket.org', 'topsecret.server.com']
>>> 'bitbucket.org' in config
True
>>> 'bytebong.com' in config
False
>>> config['bitbucket.org']['User']
'hg'
>>> config['DEFAULT']['Compression']
'yes'
>>> topsecret = config['topsecret.server.com']
>>> topsecret['ForwardX11']
'no'
>>> topsecret['Port']
'50022'
>>> for key in config['bitbucket.org']:  
...     print(key)
user
compressionlevel
serveraliveinterval
compression
forwardx11
>>> config['bitbucket.org']['ForwardX11']
'yes'
```

`注意：`[default]是供所有section共用的部分，可以理解为基类，其他的section都继承了该基础类

# 支持的数据类型

`ini文件是以string数据格式存储，所以在使用时，要按照所需进行数据的强转`

- 对于int、float可以使用int()  float()进行强转

  ```python
  >>> int(topsecret['Port'])
  50022
  >>> float(topsecret['CompressionLevel'])
  9.0
  
  ```

- 对于bool类型，如果是False，使用bool('False')得到的结果却是True

- 解决2的方法是：getboolean()方法，该方法对应的默认True|False对有'yes'`/`'no'`, `'on'`/`'off'`, `'true'`/`'false'` and `'1'`/`'0'；除此之外还提供了getint() and getfloat() 方法。也可以自己定义转换方法

  ```python
  >>> topsecret.getboolean('ForwardX11')
  False
  >>> config['bitbucket.org'].getboolean('ForwardX11')
  True
  >>> config.getboolean('bitbucket.org', 'Compression')
  True
  ```

# 返回值

- 可以使用.get()得到值，如果要设置默认值，可以使用get(key,defaultValue)

  ```python
  >>> topsecret.get('Port')
  '50022'
  >>> topsecret.get('CompressionLevel')
  '9'
  >>> topsecret.get('Cipher')
  >>> topsecret.get('Cipher', '3des-cbc')
  '3des-cbc'
  >>> topsecret.get('CompressionLevel', '3')
  '9'
  >>> config.get('bitbucket.org', 'monster',
  ...            fallback='No such things as monsters')
  'No such things as monsters'
  ```

  

- getint(), getfloat() and getboolean() 

  ```python
  >>> 'BatchMode' in topsecret
  False
  >>> topsecret.getboolean('BatchMode', fallback=True)
  True
  >>> config['DEFAULT']['BatchMode'] = 'no'
  >>> topsecret.getboolean('BatchMode', fallback=True)
  False
  ```

  # ini文件支持的格式

  ```ini
  [Simple Values]
  key=value
  spaces in keys=allowed
  spaces in values=allowed as well
  spaces around the delimiter = obviously
  you can also use : to delimit keys from values
  
  [All Values Are Strings]
  values like this: 1000000
  or this: 3.14159265359
  are they treated as numbers? : no
  integers, floats and booleans are held as: strings
  can use the API to get converted values directly: true
  
  [Multiline Values]
  chorus: I'm a lumberjack, and I'm okay
      I sleep all night and I work all day
  
  [No Values]
  key_without_value
  empty string value here =
  
  [You can use comments]
  # like this
  ; or this
  
  # By default only in an empty line.
  # Inline comments can be harmful because they prevent users
  # from using the delimiting characters as parts of values.
  # That being said, this can be customized.
  
      [Sections Can Be Indented]
          can_values_be_as_well = True
          does_that_mean_anything_special = False
          purpose = formatting for readability
          multiline_values = are
              handled just fine as
              long as they are indented
              deeper than the first line
              of a value
          # Did I mention we can indent comments, too?
  ```

  # 值插值（Interpolation of values）

  - 类configparser.BasicInterpolation

    > 只能引用同级或者默认[default]里面的值

    ```ini
    [Paths]
    home_dir: /Users
    my_dir: ${home_dir}/lumberjack
    my_pictures: ${my_dir}/Pictures
    
    [Escape]
    cost: $$80  # use a $$ to escape the $ sign ($ is the only character that needs to be escaped)
    ```

    

  - 类configparser.ExtendedInterpolation

    > 更高级的值补充，可以支持跨级，例如${section:option}

    ```ini
    [Common]
    home_dir: /Users
    library_dir: /Library
    system_dir: /System
    macports_dir: /opt/local
    
    [Frameworks]
    Python: 3.2
    path: ${Common:system_dir}/Library/Frameworks/
    
    [Arthur]
    nickname: Two Sheds
    last_name: Jackson
    my_dir: ${Common:home_dir}/twosheds
    my_pictures: ${my_dir}/Pictures
    python_dir: ${Frameworks:path}/Python/Versions/${Frameworks:Python}
    ```

  # 映射协议访问

  > 类似字典，但有一些差异：
  >
  > - 默认情况下，可以不区分大小写的方式访问节中的所有键
  > - 所有节也都包含`DEFAULTSECT`值，这意味着 `.clear()`在节上可能不会使该节明显为空。这是因为无法从该部分中删除默认值（因为从技术上讲它们不存在）。如果它们在本节中被覆盖，则删除将使默认值再次可见。尝试删除默认值会导致[`KeyError`](https://docs.python.org/3/library/exceptions.html#KeyError)。
  > - `DEFAULTSECT` 无法从解析器中删除：
  >   - 试图删除它引发[`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError)，
  >   - `parser.clear()` 保持原样，
  >   - `parser.popitem()` 永不退还。
  > - `parser.get(section, option, **kwargs)`-第二个参数**不是** 后备值。但是请注意，节级别的`get()`方法与映射协议和经典的configparser API都兼容。
  > - `parser.items()`与映射协议兼容（返回*section_name*和*section_proxy*对的列表， 包括DEFAULTSECT）。但是，此方法也可以使用参数调用。后者调用返回的列表*选项*，*值*在指定的对，与膨胀的所有内插（除非 被提供）。`parser.items(section, raw, vars)``section``raw=True`

# ConfigParser对象

```python
configparser.ConfigParser(defaults=None, dict_type=dict, allow_no_value=False, delimiters=('=', ':'), comment_prefixes=('#', ';'), inline_comment_prefixes=None, strict=True, empty_lines_in_values=True, default_section=configparser.DEFAULTSECT, interpolation=BasicInterpolation(), converters={})
# 主要配置解析器。当默认给出，它被初始化到内在默认的字典。当dict_type给出，它将被用来创建字典对象的部分名单，对于部分中的选项，并为默认值。

当分隔符给出，它被用作一组，从值分割密钥的子串。当comment_prefixes给出，它将被用作集前缀，否则空行注释子的。注释可以缩进。当inline_comment_prefixes给出，它将被用作集子的，在非空行前缀的意见。

如果使用strict为True默认值（默认设置），则解析器在从单个源（文件，字符串或字典）读取，提升DuplicateSectionError或时 将不允许任何节或选项重复DuplicateOptionError。当empty_lines_in_values为False （默认值：）时True，每个空行都标记一个选项的结尾。否则，多行选项的内部空行将保留为值的一部分。当allow_no_value为True（默认值：）时False，接受不带值的选项；否则为0。None它们所保持的值是，并且它们被序列化而没有尾随定界符。

当default_section给出，它指定为其他部分和插值目的的特殊部分保持默认值的名称（通常命名"DEFAULT"）。可以使用default_section实例属性在运行时检索和更改此值。

可以通过通过插值参数提供自定义处理程序来定制插值行为。None可用于完全关闭插值，ExtendedInterpolation()提供了受启发的更高级的变体zc.buildout。有关专用主题的更多信息，请参见 专用文档部分。

插值中使用的所有选项名称都将通过该optionxform()方法传递， 就像其他任何选项名称引用一样。例如，使用默认实现optionxform()（将选项名称转换为小写），值和等效。foo %(bar)sfoo %(BAR)s

当转换器给出，它应该是一个字典，其中每个键表示一个类型的转换器的名称和每一个值是一个可调用的执行从字符串中的转化为期望的数据类型。每个转换器get*()在解析器对象和部分代理上都有自己的对应方法。

在版本3.1中更改：默认dict_type为collections.OrderedDict。

在版本3.2中更改：添加了allow_no_value，定界符，comment_prefixes，strict， empty_lines_in_values，default_section和插值。

在3.5版本中改为：该转换器加入争论。

改变在3.7版本：在默认参数读取read_dict()，整个分析器提供一致的行为：非字符串键和值隐式转换为字符串。

在版本3.8中更改：默认dict_type为dict，因为它现在保留插入顺序。
```

- `defaults`（）
  返回包含实例范围默认值的字典。
  
- `sections`（）
  返回可用部分的列表；在*默认的部分*不包括在列表中。
  
- `add_section`（*节*）

  将名为*section的节*添加到实例。如果具有给定名称的部分已存在，[`DuplicateSectionError`](https://docs.python.org/3/library/configparser.html#configparser.DuplicateSectionError)则引发。如果传递了 *默认的节*名称，[`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError)则会引发。该部分的名称必须是字符串；如果没有，[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError)就提出来。*在版本3.2中进行了更改：*非字符串节名称提高了[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError)。

- `has_section`（*节*）

  指示配置中是否存在命名*部分*。该*默认部分*没有被确认。

- `options`（*节*）

  返回指定*部分中*可用选项的列表。

- `has_option`（*section*，*option* ）

  如果给定的*部分*存在并且包含给定的*option*，则返回 [`True`](https://docs.python.org/3/library/constants.html#True); 否则返回[`False`](https://docs.python.org/3/library/constants.html#False)。如果指定的 *部分*是[`None`](https://docs.python.org/3/library/constants.html#None)或为空字符串，则假定为DEFAULT。

- `read`（*filenames*，*encoding = None* ）

  尝试读取和解析可迭代的文件名，并返回已成功解析的文件名列表。

  如果*文件名*是字符串，[`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes)对象或类似 [路径的对象](https://docs.python.org/3/glossary.html#term-path-like-object)，则将其视为单个文件名。如果无法打开以*文件名*命名的*文件*，则该文件将被忽略。这样做是为了让您可以指定潜在配置文件位置的可迭代项（例如，当前目录，用户的主目录和某些系统范围的目录），并将读取可迭代项中的所有现有配置文件。

  如果不存在任何命名文件，则[`ConfigParser`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser) 实例将包含一个空数据集。需要从文件中加载初始值的应用程序应[`read_file()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read_file)在调用[`read()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read)任何可选文件之前使用加载一个或多个所需文件：

  ```
  import configparser, os
  
  config = configparser.ConfigParser()
  config.read_file(open('defaults.cfg'))
  config.read(['site.cfg', os.path.expanduser('~/.myapp.cfg')],
              encoding='cp1250')
  ```

  *新版本3.2：*该*编码*参数。以前，所有文件都是使用的默认编码读取的[`open()`](https://docs.python.org/3/library/functions.html#open)。

  *在新版本3.6.1：*该*文件名*参数接受[路径状物体](https://docs.python.org/3/glossary.html#term-path-like-object)。

  *新的版本3.7：*在*文件名*参数接受一个[`bytes`](https://docs.python.org/3/library/stdtypes.html#bytes)对象。

- `read_file`（*f*，*source = None* ）

  从*f*读取和解析配置数据，该数据必须是可迭代的产生Unicode字符串（例如，以文本模式打开的文件）。

  可选参数*source*指定要读取的文件的名称。如果未给出且*f*具有`name`属性，则该属性用于 *source*；默认值为`'<???>'`。

  *版本3.2中的新功能：*替换[`readfp()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.readfp)。

- `read_string`（*string*，*source ='<string>'* ）*源代码*）

  从字符串中解析配置数据。

  可选参数*source*指定所传递字符串的特定于上下文的名称。如果未给出，`'<string>'`则使用。通常应该是文件系统路径或URL。

  *3.2版中的新功能。*

- `read_dict`（*字典*，*source ='<dict>'* ）

  从任何提供类似dict`items()` 方法的对象中加载配置。键是部分名称，值是带有应在该部分中显示的键和值的字典。如果使用的字典类型保留顺序，则部分及其键将按顺序添加。值会自动转换为字符串。

  可选参数*source*指定所传递字典的特定于上下文的名称。如果未给出，`<dict>`则使用。

  此方法可用于在解析器之间复制状态。

  *3.2版中的新功能。*

- `get`（*section*，*option*，***，*raw = False*，*vars = None* [，*fallback* ] ）

  获取命名*部分*的*选项*值。如果提供了*vars*，则它必须是字典。该*选项*在*vars*（如果提供）， *section*和*DEFAULTSECT中*按该顺序查找。如果未找到密钥并提供*回退*，则将其用作回退值。 可以作为*后备*值提供。`None`

  `'%'`除非*raw*参数为true ，否则所有插值都会在返回值中扩展。以与该选项相同的方式查找插值键的值。

  *在版本3.2中更改：*参数*raw*，*vars*和*fallback*仅用作关键字，以保护用户避免尝试将第三个参数用作*后备*回退（尤其是在使用映射协议时）。

- `getint`（*section*，*option*，***，*raw = False*，*vars = None* [，*fallback* ] ）

  一种便利方法，用于将指定*节中*的*选项*强制为 整数。请参阅以获取*raw*，*vars*和 *fallback的说明*。[`get()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.get)

- `getfloat`（*section*，*option*，***，*raw = False*，*vars = None* [，*fallback* ] ）

  一种便利的方法，用于将指定*节中*的*选项*强制 为浮点数。请参阅以获取*raw*， *vars*和*fallback的说明*。[`get()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.get)

- `getboolean`（*section*，*option*，***，*raw = False*，*vars = None* [，*fallback* ] ）

  一种便利方法，用于将指定*部分中*的*选项*强制 为布尔值。请注意，该选项的接受值是 ，，，和，导致该方法返回，和，，，和，这导致它返回。这些字符串值以不区分大小写的方式检查。任何其他值都会导致它升高 。请参阅以获取*raw*，*vars*和 *fallback的说明*。`'1'``'yes'``'true'``'on'``True``'0'``'no'``'false'``'off'``False`[`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError)[`get()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.get)

- `items`（*raw = False*，*vars = None* ）

  `items`（*section*，*raw = False*，*vars = None* ）

  如果未提供*section*，则返回*section_name*和 *section_proxy*对的列表，包括DEFAULTSECT。

  否则，返回给定*部分中*选项的*名称*，*值*对列表。可选参数的含义与方法的含义相同 。[`get()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.get)

  *改变在3.8版本：*项目目前在*乏*不再出现在结果中。先前的行为将实际的解析器选项与为插值提供的变量混合在一起。

- `set`（*section*，*option*，*value* ）

  如果给定的部分存在，则将给定的选项设置为指定的值；否则提高[`NoSectionError`](https://docs.python.org/3/library/configparser.html#configparser.NoSectionError)。 *选项*和*值*必须是字符串；如果没有，[`TypeError`](https://docs.python.org/3/library/exceptions.html#TypeError)就提出来。

- `write`（*fileobject*，*space_around_delimiters = True* ）

  将配置的表示形式写入指定的[文件对象](https://docs.python.org/3/glossary.html#term-file-object)，该[对象](https://docs.python.org/3/glossary.html#term-file-object)必须以文本模式打开（接受字符串）。将来的[`read()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read)调用可以解析此表示。如果 *space_around_delimiters*为true，则键和值之间的分隔符将被空格包围。

- `remove_option`（*section*，*option* ）[¶](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.remove_option)

  从指定的*部分中*删除指定的*选项*。如果该部分不存在，请引发。如果该选项已存在，则返回；否则返回 。[`NoSectionError`](https://docs.python.org/3/library/configparser.html#configparser.NoSectionError)[`True`](https://docs.python.org/3/library/constants.html#True)[`False`](https://docs.python.org/3/library/constants.html#False)

- `remove_section`（*节*）

  从配置中删除指定的*部分*。如果该部分实际上存在，请返回`True`。否则返回`False`。

- `optionxform`（*选项*）

  将在输入文件中找到或由客户端代码传递的选项名称*选项*转换为应在内部结构中使用的形式。默认实现返回*option*的小写版本 ；子类可以覆盖此属性，或者客户端代码可以在实例上设置此名称的属性以影响此行为。

  您无需继承解析器的子类即可使用此方法，还可以在实例上将其设置为具有字符串参数并返回字符串的函数。`str`例如，将其设置为，将使选项名称区分大小写：

  ```
  cfgparser = ConfigParser()
  cfgparser.optionxform = str
  ```

  请注意，在读取配置文件时，将在[`optionxform()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.optionxform)调用选项名称之前删除空格。

- `readfp`（*fp*，*filename = None* ）

  *自版本3.2起不推荐使用：*[`read_file()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.read_file)改为使用。

  *在版本3.2中进行了更改：*[`readfp()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.readfp)现在在*fp上进行*迭代，而不是调用`fp.readline()`。

  对于[`readfp()`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser.readfp)使用不支持迭代的参数调用的现有代码，可以将以下生成器用作类似文件的对象的包装器：

  ```
  def readline_generator(fp):
      line = fp.readline()
      while line:
          yield line
          line = fp.readline()
  ```

  代替`parser.readfp(fp)`使用 `parser.read_file(readline_generator(fp))`。

## RawConfigParser对象

- *类*`configparser.``RawConfigParser`（*默认值= None*，*dict_type = dict*，*allow_no_value = False*，***，*分隔符=（'='*，*'：'）*，*comment_prefixes =（'＃'*，*';'）*，*inline_comment_prefixes = None*，*strict = True*，*empty_lines_in_values = True*，*default_section = configparser.DEFAULTSECT* [，*插值*] ）

  的旧版变体[`ConfigParser`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser)。它默认情况下禁用插值，并允许通过其unsafe`add_section`和`set`方法以及遗留`defaults=`关键字参数处理来使用非字符串节名称，选项名称和值。*在版本3.8中更改：*默认*dict_type*为[`dict`](https://docs.python.org/3/library/stdtypes.html#dict)，因为它现在保留插入顺序。注意 考虑[`ConfigParser`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser)改用哪个检查要在内部存储的值的类型。如果不想插值，可以使用`ConfigParser(interpolation=None)`。`add_section`（*节*）将名为*section的节*添加到实例。如果具有给定名称的部分已存在，[`DuplicateSectionError`](https://docs.python.org/3/library/configparser.html#configparser.DuplicateSectionError)则引发。如果传递了 *默认的节*名称，[`ValueError`](https://docs.python.org/3/library/exceptions.html#ValueError)则会引发。未检查*节的*类型，这使用户可以创建非字符串命名节。不支持此行为，并且可能会导致内部错误。`set`（*section*，*option*，*value* ）如果给定的部分存在，则将给定的选项设置为指定的值；否则提高[`NoSectionError`](https://docs.python.org/3/library/configparser.html#configparser.NoSectionError)。虽然可以使用 [`RawConfigParser`](https://docs.python.org/3/library/configparser.html#configparser.RawConfigParser)（或[`ConfigParser`](https://docs.python.org/3/library/configparser.html#configparser.ConfigParser)将*原始*参数设置为true）用于*内部*非字符串值的存储，但是仅使用字符串值才能实现全部功能（包括插值和输出到文件）。此方法使用户可以在内部为键分配非字符串值。不支持此行为，并且在尝试写入文件或以非原始模式获取文件时会导致错误。 **使用** 不允许进行此类分配**的映射协议API**。

## 异常

- *异常*`configparser.``Error`

  所有其他[`configparser`](https://docs.python.org/3/library/configparser.html#module-configparser)异常的基类。

- *异常*`configparser.``NoSectionError`

  未找到指定节时引发异常。

- *异常*`configparser.``DuplicateSectionError`

  如果`add_section()`在单个输入文件，字符串或字典中多次发现某个节，则使用已经存在的节的名称或在严格的解析器中调用if会引发异常。*3.2版中的新功能：*可选`source`，`lineno`并[`__init__()`](https://docs.python.org/3/reference/datamodel.html#object.__init__)添加了属性和参数 。

- *异常*`configparser.``DuplicateOptionError`

  如果从单个文件，字符串或字典中读取单个选项两次，则严格解析器会引发异常。这会捕获拼写错误和与大小写敏感有关的错误，例如，词典中可能有两个键代表同一个不区分大小写的配置键。

- *异常*`configparser.``NoOptionError`

  在指定的部分中找不到指定的选项时引发异常。

- *异常*`configparser.``InterpolationError`

  在执行字符串插值时发生问题时引发的异常的基类。

- *异常*`configparser.``InterpolationDepthError`

  由于迭代次数超过不能完成字符串插值时引发的异常[`MAX_INTERPOLATION_DEPTH`](https://docs.python.org/3/library/configparser.html#configparser.MAX_INTERPOLATION_DEPTH)。的子类 [`InterpolationError`](https://docs.python.org/3/library/configparser.html#configparser.InterpolationError)。

- *异常*`configparser.``InterpolationMissingOptionError`

  从值引用的选项不存在时引发的异常。的子类[`InterpolationError`](https://docs.python.org/3/library/configparser.html#configparser.InterpolationError)。

- *异常*`configparser.``InterpolationSyntaxError`

  当进行替换的源文本不符合要求的语法时，引发异常。的子类[`InterpolationError`](https://docs.python.org/3/library/configparser.html#configparser.InterpolationError)。

- *异常*`configparser.``MissingSectionHeaderError`

  尝试分析没有节头的文件时引发异常。

- *异常*`configparser.``ParsingError`

  尝试解析文件时发生错误时引发异常。*在版本3.2中进行了更改：*为了保持一致，将`filename`属性和[`__init__()`](https://docs.python.org/3/reference/datamodel.html#object.__init__)参数重命名 `source`为。