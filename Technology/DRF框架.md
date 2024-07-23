# DRF框架

## 〇、About

-  <span id="model">模型用例</span>

    ```python
    from django.db import models
    
    # Create your models here.
    
    class Book(models.Model):
        title = models.CharField(max_length=32,verbose_name="书籍名称")
        price = models.IntegerField(verbose_name="价格")
        pub_date = models.DateField(verbose_name="出版日期")
        
        bread = models.IntegerField(verbose_name="阅读量") 
        bcomment = models.IntegerField(verbose_name="评论量") 
        
        # publish和book  一对多关系  外键设在多的book表内
        publish = models.ForeignKey("Publish",on_delete=models.CASCADE,verbose_name="出版社")
        # 采用半自动方式建立author和book表之间的多对多关系
        authors = models.ManyToManyField(to="Author",
                                     through="Book2Author",
                                     through_fields=('book','author'),
                                     verbose_name="作者", related_name='books'
                                     )
        # 采用全自动方式建立editor和book表之间的多对多关系
        editors = models.ManyToManyField(to="Editor")
        
        
        def __str__(self):
            return self.title
    
    
    class Publish(models.Model):
        name = models.CharField(max_length=32, verbose_name="出版社名称")
        email = models.EmailField(verbose_name="出版社邮箱")
    
        def __str__(self):
            return self.name
    
    # author表,和book表多对多
    class Author(models.Model):
        name = models.CharField(max_length=32, verbose_name="作者")
        age = models.IntegerField(verbose_name="年龄")
    
        def __str__(self):
            return self.name
    
    # editor表,和book多对多
    class Editor(models.Model):
        name = models.CharField(max_length=255,verbose_name="主编姓名")
     
        class Meta:
            db_table = "bookstore_editor"
            verbose_name = "主编"
            verbose_name_plural = verbose_name
     
        def __str__(self):
            return self.name
        
    # 半自动创建一个book和author的多对多关系表
    # 和前面自动创建的book和editor的多对多关系表形成对比
    class Book2Author(models.Model):
        book = models.ForeignKey(to="Book", on_delete=models.CASCADE)
        author = models.ForeignKey(to="Author", on_delete=models.CASCADE)
     
        class Meta:
            db_table = "bookstore_book2author"
            verbose_name = "书籍作者关系表"
            verbose_name_plural = verbose_name　
    
    ```
    
    

 ## 一、web应用模式

## 二、api接口

## 三、RESTful API规范

## 四、序列化

## 五、Django Rest_Framework

## 六、环境的安装与配置

## 七、序列化器-serializer

- 作用
    1. 序列化,序列化器会把模型对象转换成字典,经过response以后变成json字符串 
    2. 反序列化,把客户端发送过来的数据,经过request以后变成字典,序列化器可以把字典转成模型 
    3. 反序列化,完成数据校验功能

### 1、序列化器的定义

- Django REST framework中的Serializer使用类来定义，须继承自`rest_framework.serializers.Serializer`

#### (1)基本方法

- 为模型类提供一个序列化器，以[Book模型](#model)为例，可以定义如下：

    ```python
    from rest_framework import serializers
    
    class BookSerializer(serializers.Serializer):
        title = serializers.CharField()
        price = serializers.IntegerField()
        pub_date = serializers.DateField()
        publish_id = serializers.IntegerField(write_only=True)
        publish_name = serializers.CharField(source="publish.name", read_only=True)
        publish_email = serializers.CharField(source="publish.email", read_only=True)
    ```

- 常用字段

    | 字段                    | 字段构造方式                                                 |
    | ----------------------- | ------------------------------------------------------------ |
    | **BooleanField**        | BooleanField()                                               |
    | **NullBooleanField**    | NullBooleanField()                                           |
    | **CharField**           | CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True) |
    | **EmailField**          | EmailField(max_length=None, min_length=None, allow_blank=False) |
    | **RegexField**          | RegexField(regex, max_length=None, min_length=None, allow_blank=False) |
    | **SlugField**           | SlugField(max_length=50, min_length=None, allow_blank=False) <br />正则字段，验证正则模式 [a-zA-Z0-9*-]+ |
    | **URLField**            | URLField(max_length=200, min_length=None, allow_blank=False) |
    | **UUIDField**           | UUIDField(format=‘hex_verbose’) <br />format: <br />1）`'hex_verbose'` 如`"5ce0e9a5-5ffa-654b-cee0-1238041fb31a"` <br />2）`'hex'` 如 `"5ce0e9a55ffa654bcee01238041fb31a"` <br />3）`'int'`如: `"123456789012312313134124512351145145114"` <br />4）`'urn'` 如: `"urn:uuid:5ce0e9a5-5ffa-654b-cee0-1238041fb31a"` |
    | **IPAddressField**      | IPAddressField(protocol=‘both’, unpack_ipv4=False, **options) |
    | **IntegerField**        | IntegerField(max_value=None, min_value=None)                 |
    | **FloatField**          | FloatField(max_value=None, min_value=None)                   |
    | **DecimalField**        | DecimalField(max_digits, decimal_places, coerce_to_string=None, max_value=None, min_value=None) max_digits: 最多位数 decimal_palces: 小数点位置 |
    | **DateTimeField**       | DateTimeField(format=api_settings.DATETIME_FORMAT, input_formats=None) |
    | **DateField**           | DateField(format=api_settings.DATE_FORMAT, input_formats=None) |
    | **TimeField**           | TimeField(format=api_settings.TIME_FORMAT, input_formats=None) |
    | **DurationField**       | DurationField()                                              |
    | **ChoiceField**         | ChoiceField(choices) choices与Django的用法相同               |
    | **MultipleChoiceField** | MultipleChoiceField(choices)                                 |
    | **FileField**           | FileField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL) |
    | **ImageField**          | ImageField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL) |
    | **ListField**           | ListField(child=, min_length=None, max_length=None)          |
    | **DictField**           | DictField(child=)                                            |

- 选项参数

    | 参数名称            | 作用             |
    | ------------------- | ---------------- |
    | **max_length**      | 最大长度         |
    | **min_lenght**      | 最小长度         |
    | **allow_blank**     | 是否允许为空     |
    | **trim_whitespace** | 是否截断空白字符 |
    | **max_value**       | 最小值           |
    | **min_value**       | 最大值           |

- 通用参数：

    | 参数名称           | 说明                                          |
    | ------------------ | --------------------------------------------- |
    | **read_only**      | 表明该字段仅用于序列化输出，默认False         |
    | **write_only**     | 表明该字段仅用于反序列化输入，默认False       |
    | **required**       | 表明该字段在反序列化时必须输入，默认True      |
    | **default**        | 反序列化时使用的默认值                        |
    | **allow_null**     | 表明该字段是否允许传入None，默认False         |
    | **validators**     | 该字段使用的验证器                            |
    | **error_messages** | 包含错误编号与错误信息的字典                  |
    | **label**          | 用于HTML展示API页面时，显示的字段名称         |
    | **help_text**      | 用于HTML展示API页面时，显示的字段帮助提示信息 |

#### (2)ModelSerializer

- 如果我们想要使用序列化器对应的是Django的模型类，DRF为我们提供了ModelSerializer模型类序列化器来帮助我们快速创建一个Serializer类。

- ModelSerializer与常规的Serializer相同，但提供了：

    - 基于模型类自动生成一系列字段

    - 基于模型类自动为Serializer生成validators，比如unique_together

    - 包含默认的create()和update()的实现

##### a.定义

- 比如我们以[Book模型](#model)为例，创建一个BookSerializer

    ```python
    from rest_framework import serializers
    
    class BookSerializer(serializers.ModelSerializer):
        """图书数据序列化器"""
        class Meta:
            model = Book  # 指明参照哪个模型类
            fields = '__all__'  # 指明为模型类的哪些字段生成
    ```

    我们可以在python manage.py shell中查看自动生成的BookSerializer的具体实现

    ```python
    >>> from sers.sers import BookSerializer
    >>> serializer = BookSerializer()
    >>> serializer
    BookSerializer():
        id = IntegerField(label='ID', read_only=True)
        title = CharField(label='书籍名称', max_length=32)
        price = IntegerField(label='价格')
        pub_date = DateField(label='出版日期')
        bread = IntegerField(required=False)
        bcomment = IntegerField(required=False)
        publish = PrimaryKeyRelatedField(label='出版社', queryset=Publish.objects.all())
        authors = PrimaryKeyRelatedField(allow_empty=False, label='作者', many=True, queryset=Author.objects.all())
    >>>
    ```

- 如果希望特殊的设置某个字段可以直接在类中进行添加（修改）

    ```python
    from rest_framework import serializers
    from author_serializer import AuthorSerizlizer
    
    class BookSerializer(serializers.ModelSerializer):
        authors = AuthorSerizlizer(read_only=True, many=True)
        
        """图书数据序列化器"""
        class Meta:
            model = Book  # 指明参照哪个模型类
            fields = '__all__'  # 指明为模型类的哪些字段生成
    ```

    此时这个类相当于（可以对照上文的自动生成的BookSerializer的具体实现阅读）

    ```python
    class BookSerializer():
        id = IntegerField(label='ID', read_only=True)
        title = CharField(label='书籍名称', max_length=32)
        price = IntegerField(label='价格')
        pub_date = DateField(label='出版日期')
        bread = IntegerField(required=False)
        bcomment = IntegerField(required=False)
        publish = PrimaryKeyRelatedField(label='出版社', queryset=Publish.objects.all())
        authors = AuthorSerizlizer(read_only=True, many=True)
    ```

##### b.字段设置

- 使用 **fields** 来明确字段，`__all__` 表名包含所有字段，也可以写明具体哪些字段

    ```python
    class BookSerializer(serializers.ModelSerializer):
        """图书数据序列化器"""
        class Meta
            model = BookInfo
            fields = ('id', 'title', 'pub_date')
    ```

- 使用**exclude**可以明确排除掉哪些字段

    ```python
    class BookSerializer(serializers.ModelSerializer):
        """图书数据序列化器"""
        class Meta:
            model = Book
            fields = '__all__'  # 新版本的drf貌似必须要这个才可以用exculde，待考证
            exclude = ('pub_date',)
    ```

- 可以通过**read_only_fields**指明只读字段，即仅用于序列化输出的字段

    ```python
    class BookSerializer(serializers.ModelSerializer):
        """图书数据序列化器"""
        class Meta:
            model = Book
            fields = ('id', 'title', 'pub_date'， 'bread', 'bcomment')
            read_only_fields = ('id', 'bread', 'bcomment')
    ```

- 可以使用**extra_kwargs**参数为ModelSerializer添加或修改原有的选项参数

    ```python
    class BookSerializer(serializers.ModelSerializer):
        """图书数据序列化器"""
        class Meta:
            model = Book
            fields = ('id', 'title', 'pub_date', 'bread', 'bcomment')
            extra_kwargs = {
                'bread': {'min_value': 0, 'required': True},
                'bcomment': {'min_value': 0, 'required': True},
            }
    # 相当于
    class BookSerializer():
        id = IntegerField(label='ID', read_only=True)
        title = CharField(label='书籍名称', max_length=32)
        pub_date = DateField(label='出版日期')
        bread = IntegerField(min_value=0, required=True)
        bcomment = IntegerField(min_value=0, required=True)
    ```
    
    Django REST framework 还提供了其他一些可用于 `extra_kwargs` 的关键字参数，例如：
    
    - `allow_blank`: 对于字符串类型的字段，指定是否允许输入空白字符串。
    - `allow_null`: 指定字段是否允许为 `None` 值。
    - `default`: 指定当没有提供字段值时，该字段应该使用的默认值。
    - `min_value` 和 `max_value`: 对于数字类型的字段，指定最小值和最大值。
    - `min_length` 和 `max_length`: 对于字符串类型的字段，指定字符串的最小长度和最大长度。
    - `style`: 指定用于呈现该字段的样式。

#### (3)关联字段

- **PrimaryKeyRelatedField**

    此字段将被序列化为关联对象的主键。

    ```python
    hbook = serializers.PrimaryKeyRelatedField(label='图书', read_only=True)
    # or
    hbook = serializers.PrimaryKeyRelatedField(label='图书', queryset=BookInfo.objects.all())
    ```

    指明字段时需要包含read_only=True或者queryset参数：

    - 包含read_only=True参数时，该字段将不能用作反序列化使用
    - 包含queryset参数时，将被用作反序列化时参数校验使用

- **StringRelatedField**

    此字段将被序列化为关联对象的字符串表示方式（即`__str__`方法的返回值）

    ```python
    hbook = serializers.StringRelatedField(label='图书')
    ```

- **HyperlinkedRelatedField**

    此字段将被序列化为获取关联对象数据的接口链接

    ```python
    hbook = serializers.HyperlinkedRelatedField(label='图书', read_only=True, view_name='books-detail')
    ```

    必须指明view_name参数，以便DRF根据视图名称寻找路由，进而拼接成完整URL

- **SlugRelatedField**

    此字段将被序列化为关联对象的指定字段数据

    ```python
    hbook = serializers.SlugRelatedField(label='图书', read_only=True, slug_field='bpub_date')
    ```

    slug_field指明使用关联对象的哪个字段

- **使用关联对象的序列化器**

    这个我认为最好用，详见[嵌套序列化](####(4)嵌套序列化)

- **重写to_representation方法**

    序列化器的每个字段实际都是由该字段类型的to_representation方法决定格式的，可以通过重写该方法来决定格式

    > 注意，to_representations方法不仅局限在控制关联对象格式上，适用于各个序列化器字段类型。

    自定义新的关联字段：

    ```python
    class BookRelatedPublish(serializers.RelatedField):
    
        def to_representation(self, value):
            return "pub-%s" % value.name
    
    class BookRelatedAuthor(serializers.RelatedField):
    
        def to_representation(self, value):
            return "author-%s" % value.name
    ```

    使用声明的关联字段

    ```python
    class BookSerializer(serializers.Serializer):
        title = serializers.CharField(max_length=32)
        price = serializers.IntegerField(required=False)
        pub_date = serializers.DateField(required=False)
        # 使用声明的关联字段
        authors = BookRelatedAuthor(read_only=True,many=True)
        publish = BookRelatedPublish(read_only=True)
    ```

- 关于many属性

    如果关联的对象数据不是只有一个，而是包含多个数据，此时关联字段类型的指明仍可使用上述几种方式，只是在声明关联字段时，多补充一个 **many=True** 参数即可

#### (4)嵌套序列化

- 不管是自动化创建的多对多关系，还是半自动创建的多对多关系，以及一对多关系，使用depth设置都能够正确的进行序列化信息显示。但是，一旦使用了含有序列化深度depth属性的序列化器，那么引用改属性的字段就无法进行更新或者创建

    ```python
    class BookSerializer(serializers.ModelSerializer):
     
        class Meta:
            model = models.Book
            fields = "__all__"
            depth = 1
    # 序列化深度depth属性能够将关联属性的表属性也显示序列化出来
    # 引用序列化深度depth属性的字段序列化时会自动变为readonly,所以更新或创建时,不能使用该序列化器类
    ```

- 正向查找和序列化

  当外键和多对多关系在本模型类中声明时，可以用如下方法书写序列化器
  
  另外为同时满足一对多,多对多创建更新的需求，需要重写`create()`方法和`update()`方法
  
    ```python
    class BookSerializer(serializers.ModelSerializer):
        # 嵌套序列化信息
        editors = EditorModelSerializer(many=True)
        authors = AuthorModelSerializer(many=True)
        publish = PublishModelSerializer()
     
        class Meta:
            model = models.Book
            fields = "__all__"
     
        def create(self, validated_data):
            editors_data = validated_data.pop('editors')
            authors_data = validated_data.pop('authors')
            publish_data = validated_data.pop('publish')
            publish = models.Publish.objects.get(name=publish_data.get('name'))
            # 用fliter就是queryset,用get才是对象instance或obj
            # 添加一对多数据,和orm用法一致
            book = models.Book.objects.create(**validated_data, publish=publish)
            # 添加多对多字段数据,同样和orm用法一致
            for editor_data in editors_data:
                editor = models.Editor.objects.get(name=editor_data.get('name'))
                book.editors.add(editor)
            # 添加多对多字段数据,此处是半自动创建的多对多,无法使用add方法,用orm的create方法
            for author_data in authors_data:
                author = models.Author.objects.get(name=author_data.get('name'))
                models.Book2Author.objects.create(book=book, author=author)
            return book
       	
        def update(self, validated_data):
            ... ...
    ```
  
- 反向查找和序列化

    上述一对多，多对多序列化我们是从外键所在的表正向查找另外的表的数据,再进行序列化，如果我们想从一对多的一的一方反向查找多的一方并序列化，或者从多对多的非外键所在表反向查找

    ```python
    # editor表和book表示多对多,外键在book表中
    # publish表和book表示一对多,外键在book表中
     
    class EditorModelSerializer(serializers.ModelSerializer):
        # 反向查找
        book_set = BookSerializer(many=True,read_only=True)
        # book_set = serializers.PrimaryKeyRelatedField(many=True,read_only=True)
        class Meta:
            model = models.Editor
            # fields = ['name']
            fields = "__all__"
     
     
    class PublishModelSerializer(serializers.ModelSerializer):
        # 反向查询
        # book_set = serializers.StringRelatedField(many=True,read_only=True)
        book_set = BookSerializer(many=True,read_only=True)
        class Meta:
            model = models.Publish
            fields = "__all__"
     
    class AuthorModelSerializer(serializers.ModelSerializer):
        # 反向查询
        book_set = BookSerializer(many=True,read_only=True)
        class Meta:
            model = models.Author
            fields = "__all__"
    
    # 使用orm语法的标识:"表名_set"即可反向查找
    # 对自动创建的多对多和半自动创建的多对多同样适用
    ```

- 关于**related_name**

    `related_name` 是 Django 的 `ManyToManyField` 、`ForeignKey`属性之一，用于指定从目标模型回到源模型的反向关系的名称。它允许程序员指定反向关系的自定义名称，而不是使用默认名称，该默认名称是模型名称的小写形式，后面加上 `_set`。

    例如，如果有一个名为 `Author` 的模型和另一个名为 `Book` 的模型，并且想在它们之间定义多对多关系，则可以在 `Author` 模型中使用 `ManyToManyField`，如下所示：

    ```python
    class Author(models.Model):
        name = models.CharField(max_length=100)
        books = models.ManyToManyField(Book, related_name='authors')
    ```

    在这里，我们使用 `related_name='authors'` 指定了反向关系的自定义名称。这意味着如果您有一个 `Book` 实例，则可以使用属性 `authors` 而不是默认属性 `book_set` 访问其所有作者。

### 2、创建序列化器对象

- Serializer的构造方法为：

    ```python
    Serializer(instance=None, data=empty, **kwarg)
    ```

    1. 用于序列化时，将模型类对象传入**instance**参数

    2. 用于反序列化时，将要被反序列化的数据传入**data**参数

    3. 除了instance和data参数外，在构造Serializer对象时，还可通过**context**参数额外添加数据，如

        ```python
        serializer = AccountSerializer(account, context={'request': request})
        ```

        通过context参数附加的数据，可以通过Serializer对象的context属性获取。

- tips

    > 1. 使用序列化器的时候一定要注意，序列化器声明了以后，不会自动执行，需要我们在视图中进行调用才可以
    > 2. 序列化器无法直接接收数据，需要我们在视图中创建序列化器对象时把使用的数据传递过来
    > 3. 序列化器的字段声明类似于我们前面使用过的表单系统
    > 4. 开发restful api时，序列化器会帮我们把模型数据转换成字典
    > 5. drf提供的视图会帮我们把字典转换成json,或者把客户端发送过来的数据转换字典

### 3、序列化器的使用

- 序列化器的使用分两个阶段：

    1. 处理服务器响应时，使用序列化器可以完成对数据的序列化。

    2. 处理客户端请求时，使用序列化器可以完成对数据的反序列化。

#### (1)序列化

- 基本流程

    ```python
    from rest_framework.response import Response
    from rest_framework.views import APIView
    from .models import Book
    from .sers import BookSerializer
    
    class BookView(APIView):
    
        def get(self, request):
            # 先查询出一个学生对象
            book = Book.objects.get(pk=1)
            # 构造序列化器对象
            bs = BookSerializer(instance=book)
            # 获取序列化数据bs.data
            return Response(bs.data)
    ```

- 如果要被序列化的是包含多条数据的查询集QuerySet，可以通过添加**many=True**参数补充说明

    ```python
    class BookView(APIView):
    
        def get(self, request):
            # book = Book.objects.get(pk=1)
            books = Book.objects.all()
            bs = BookSerializer(instance=books, many=True)
            return Response(bs.data)
    ```

#### (2)反序列化

##### a.数据校验

- 使用序列化器进行反序列化时，需要对数据进行验证后，才能获取验证成功的数据或保存成模型类对象。

- 使用**is_valid()**函数

    - 在获取反序列化的数据前，必须调用**is_valid()**方法进行验证，验证成功返回True，否则返回False。

    - 验证失败，可以通过序列化器对象的**errors**属性获取错误信息，返回字典，包含了字段和字段的错误。如果是非字段错误，可以通过修改REST framework配置中的**NON_FIELD_ERRORS_KEY**来控制错误字典中的键名。

    - 验证成功，可以通过序列化器对象的**validated_data**属性获取数据

        ```python
        >>>from sers.sers import BookSerializer
        >>>bs = BookSerializer(data={"title":"小王子","price":100})
        >>>bs.is_valid()  # 必须先要is_valid,才会有bs.validated_data和bs.errors
        False
        >>>bs.validated_data
        {}
        >>>bs.errors
        {'pub_date': [ErrorDetail(string='This field is required.', code='required')], 'publish_id': [ErrorDetail(string='This field is required.', code='required')]}
        ```

    - 在定义序列化器时，指明每个字段的序列化类型和选项参数，本身就是一种验证行为。通过构造序列化器对象，并将要反序列化的数据传递给data构造参数，进而进行验证

        > 可以设置required=False让校验字段可以为空！

    - is_valid()方法还可以在验证失败时抛出异常serializers.ValidationError，可以通过传递**raise_exception=True**参数开启，REST framework接收到此异常，会向前端返回HTTP 400 Bad Request响应。

- 自定义验证

    - validate_<field_name>

        对`<field_name>`字段进行验证

        ```python
        class BookSerializer(serializers.Serializer):
            title = serializers.CharField(max_length=32)
            price = serializers.IntegerField(required=True)
            pub_date = serializers.DateField(required=True)
        
            def validate_title(self, value):
                if 'django' not in value.lower():
                    raise serializers.ValidationError("图书不是关于Django的")
                return value
        
        # test
        >>>from sers.sers import BookSerializer
        >>>bs = BookSerializer(data={"title":"小王子","price":100})
        >>>bs.is_valid()
        False
        >>>bs.errors
        {'title': [ErrorDetail(string='图书不是关于Django的', code='invalid')], 'pub_date': [ErrorDetail(string='This field is required.', code='required')]}
        ```

        还有一种写法

        ```python
        def title_django(self, value):
            if 'django' not in value.lower():
                raise serializers.ValidationError("图书不是关于Django的")
            return value
        
        class BookSerializer(serializers.Serializer):
            title = serializers.CharField(max_length=32,validators=[title_django,])
            ...
        ```

    - validate

        在序列化器中需要同时对多个字段进行验证时（通常这些字段有一定的关联），可以定义`validate`方法来验证

        ```python
        class BookSerializer(serializers.Serializer):
            title = serializers.CharField(max_length=32)
            price = serializers.IntegerField(required=False)
            pub_date = serializers.DateField(required=False)
        
            bread = serializers.IntegerField(label='阅读量', required=False)
            bcomment = serializers.IntegerField(label='评论量', required=False)
        
        
            def validate_title(self, value):
                if 'django' not in value.lower():
                    raise serializers.ValidationError("图书不是关于Django的")
                return value
        
            def validate(self, data):
                bread = data.get("bread")
                bcomment = data.get("bcomment")
                if bread < bcomment:
                    raise serializers.ValidationError('阅读量小于评论量')
                return data
            
        # test
        >>>bs = BookSerializer(data={"title":"Django深入浅出","bread":100,"bcomment":200,"publish_id":1})
        >>>bs.is_valid()
        False
        >>>bs.errors
        {'non_field_errors': [ErrorDetail(string='阅读量小于评论量', code='invalid')]}
        ```

##### b.反序列化-保存数据

- 验证数据成功后，我们可以使用序列化器来完成数据反序列化的过程，这个过程可以把数据转成模型类对象

- 可以通过实现`create()`和`update()`两个方法来实现

    ```python
    class BookSerializer(serializers.Serializer):
        """图书数据序列化器"""
        ...
    
        def create(self, validated_data):
            """新建"""
            return instance
    
        def update(self, instance, validated_data):
            """更新"""
            return instance
    ```

    实现了上述两个方法后，在反序列化数据的时候，就可以通过save()方法返回一个数据对象实例了

    ```
    book = serializer.save()
    ```

    > 1、如果创建序列化器对象的时候，没有传递instance实例，则调用save()方法的时候，create()被调用
    >
    > 2、相反，如果传递了instance实例，则调用save()方法的时候，update()被调用。

##### c.附加说明

1. 在对序列化器进行save()保存时，可以额外传递数据，这些数据可以在create()和update()中的validated_data参数获取到

    ```python
    # request.user 是django中记录当前登录用户的模型对象
    book = serializer.save(owner=request.user)
    ```

2. 默认序列化器必须传递所有required的字段，否则会抛出验证异常。但是我们可以使用partial参数来允许部分字段更新

    ```python
    # Update `comment` with partial data
    serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)
    ```

3. `to_internal_value`函数

    `to_internal_value`方法是序列化器中定义的一个函数。

    当您提交数据到 API 端点时，输入数据被序列化为 Python 对象，并被验证以确保其有效性。`to_internal_value` 方法被用于将输入数据转换为 Python 对象，并且通常被用于在执行验证之前转换数据类型或对数据进行预处理。

    因此，`to_internal_value` 方法可以说是 DRF 中序列化器的一个核心函数，它允许我们对输入数据进行处理和验证，以确保它符合我们的要求。

## 八、视图

- Django REST framwork 提供的视图的主要作用：

    - 控制序列化器的执行（检验、保存、转换数据）
    - 控制数据库模型的操作

    REST framework 提供了众多的通用视图基类与扩展类，以简化视图的编写

### 1、两个视图基类

#### (1)APIView[基本视图类]

- `APIView`是REST framework提供的所有视图的基类，继承自Django的`View`父类。

- `APIView`与`View`的不同之处在于：

    - 传入到视图方法中的是REST framework的`Request`对象，而不是Django的`HttpRequeset`对象；

    - 视图方法可以返回REST framework的`Response`对象，视图会为响应数据设置（render）符合前端期望要求的格式；

    - 任何`APIException`异常都会被捕获到，并且处理成合适格式的响应信息返回给客户端；

    - 重新声明了一个新的as_views方法并在dispatch()进行路由分发前，会对请求的客户端进行身份认证、权限检查、流量控制。

- `APIView`新增了类属性

    - **authentication_classes** 列表或元组，身份认证类

    - **permissoin_classes** 列表或元组，权限检查类

    - **throttle_classes** 列表或元祖，流量控制类

- 在`APIView`中仍以常规的类视图定义方法来实现get() 、post() 或者其他请求方式的方法。

- drf在django原有的基础上，新增了一个request对象，并在django原有的HttpResponse响应类的基础上实现了一个子类rest_framework.response.Response响应类。这两个类，都是基于内容协商来完成数据的格式转换的。

    > request->parser->识别客户端请求头中的Content-Type来完成数据转换成类字典(QueryDict，字典的子类)
    >
    > response->renderer->识别客户端请求头的Accept来提取客户单期望的返回数据格式，转换成客户端的期望格式数据

##### a.请求

- REST framework 传入视图的request对象不再是Django默认的HttpRequest对象，而是REST framework提供的扩展了HttpRequest类的**Request**类的对象。

- REST framework 提供了**Parser**解析器，在接收到请求后会自动根据Content-Type指明的请求数据类型（如JSON、表单等）将请求数据进行parse解析，解析为类字典[QueryDict]对象保存到**Request**对象中。

- **Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。**

- 无论前端发送的哪种格式的数据，我们都可以以**统一的方式读取数据**。

###### (a)常用属性

1. data

    `request.data` 返回解析之后的请求体数据。类似于Django中标准的`request.POST`和 `request.FILES`属性，但提供如下特性：

    - 包含了解析之后的文件和非文件数据
    - 包含了对POST、PUT、PATCH请求方式解析后的数据
    - 利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据

2. query_params

    `request.query_params`与Django标准的`request.GET`相同，只是更换了更正确的名称而已。

3. request._request

    获取django封装的Request对象，即HttpRequest类型的Request对象

###### (b)基本使用

- ```python
    from django.views import View
    from django.http.response import HttpResponse
    from django.http.request import HttpRequest
    from django.core.handlers.wsgi import WSGIRequest
    class ReqView(View):
        def get(self,request):
            print(request)
            return HttpResponse("ok")
    
    """
    默认情况下, 编写视图类时，如果继承的是django内置的django.view.View视图基类，
    则视图方法中得到的request对象，是django默认提供的django.core.handlers.wsgi.WSGIRequest
    WSGIRequest这个请求处理对象，无法直接提供的关于json数据数据处理。
    在编写api接口时很不方便，所以drf为了简写这块内容，在原来的HttpRequest的基础上面，新增了一个Request对象
    这个Request对象是单独声明的和原来django的HttpRequest不是父子关系。
    同时注意：
       要使用drf提供的Request请求处理对象，必须在编写视图类时继承drf提供的视图基类
       from rest_framework.views import APIView
       
       如果使用drf提供的视图基类APIView编写类视图，则必须使用来自drf提供的Request请求对象和Response响应对象
    """
    from rest_framework.views import APIView
    from rest_framework.response import Response
    from rest_framework import status
    class ReqAPIView(APIView):
        def get(self,request):
            # rest_framework.request.Request对象
            # <rest_framework.request.Request: GET '/req/req2?name=xiaoming&age=17&lve=swim&lve=code'>
            print(request) 
            # 获取查询字符串
            print(request.query_params)
            # 没有参数情况下： <QueryDict: {}>
            # 有参数的情况下： <QueryDict: {'name': ['xiaoming'], 'age': ['17'], 'lve': ['swim', 'code']}>
            # 所以，request.query_params的返回值操作和原来在django里面是一模一样的
            print(request.query_params.get("name")) # xiaoming
            print(request.query_params.getlist("lve")) # ['swim', 'code']
    
            return Response("ok")
    
        def post(self, request):
            # 获取请求体
            print(request.data) # {'name': 'xiaoming', 'age': 16, 'lve': ['swim', 'code']}
            """直接从请求体中提取数据转
            # 客户端如果上传了json数据，直接返回字典
            {'name': '灰太狼', 'age': 20, 'sex': 1, 'classmate': '301', 'description': '我还会再回来的~'}
            # 客户端如果上传了表单数据，直接返回QueryDict
            <QueryDict: {'name': ['xiaohui'], 'age': ['18']}>
            """
            print(request.FILES) # 获取上传文件列表
            # 要获取django原生提供的HttpRequest对象，可以通过request._request来获取到
            print(request._request.META.get("Accept")) # 当值为None时，drf默认在响应数据时按json格式返回
            # response = Response(data="not ok", status=204, headers={"Company":"Oldboy"})
            response = Response(data="not ok", status=status.HTTP_400_BAD_REQUEST, headers={"Company":"Oldboy"})
            return response
    ```

##### b.响应

- REST framework提供了一个响应类`Response`，使用该类构造响应对象时，响应的具体数据内容会被转换（render渲染器）成符合前端需求的类型。

- REST framework提供了`Renderer` 渲染器，用来根据请求头中的`Accept`（接收数据类型声明）来自动转换响应数据到对应格式。如果前端请求中未进行Accept声明，则会采用Content-Type方式处理响应数据，我们可以通过配置来修改默认响应格式。

- 可以在**rest_framework.settings**查找所有的drf默认配置项

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
            'rest_framework.renderers.JSONRenderer',  # json渲染器，返回json数据
            'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览器API渲染器，返回调试界面
        )
    }
    ```

###### (a)构造方式

- ```python
    Response(data, status=None, template_name=None, headers=None, content_type=None)
    ```

    > - `data`: 为响应准备的序列化处理后的数据；
    > - `status`: 状态码，默认200；
    > - `template_name`: 模板名称，如果使用`HTMLRenderer` 时需指明；
    > - `headers`: 用于存放响应头信息的字典；
    > - `content_type`: 响应数据的Content-Type，通常此参数无需传递，REST framework会根据前端所需类型数据来设置该参数

    - drf的响应处理类和请求处理类不一样，Response就是django的HttpResponse响应处理类的子类。

    - `data`数据不要是render处理之后的数据，只需传递python的内建类型数据即可，REST framework会使用`renderer`渲染器处理`data`。

    - `data`不能是复杂结构的数据，如Django的模型类对象，对于这样的数据我们可以使用`Serializer`序列化器序列化处理后（转为了Python字典类型）再传递给`data`参数。

###### (b)response对象的属性

- > data：传给response对象的序列化后，但尚未render处理的数据
    >
    > status_code：状态码的数字
    >
    > content：经过render处理后的响应数据

###### (c)状态码

- 为了方便设置状态码，REST framewrok在`rest_framework.status`模块中提供了常用http状态码的常量。

    ```python
    # 1）信息告知 - 1xx
    HTTP_100_CONTINUE
    HTTP_101_SWITCHING_PROTOCOLS
    
    # 2）成功 - 2xx
    HTTP_200_OK
    HTTP_201_CREATED
    HTTP_202_ACCEPTED
    HTTP_203_NON_AUTHORITATIVE_INFORMATION
    HTTP_204_NO_CONTENT
    HTTP_205_RESET_CONTENT
    HTTP_206_PARTIAL_CONTENT
    HTTP_207_MULTI_STATUS
    
    # 3）重定向 - 3xx
    HTTP_300_MULTIPLE_CHOICES
    HTTP_301_MOVED_PERMANENTLY
    HTTP_302_FOUND
    HTTP_303_SEE_OTHER
    HTTP_304_NOT_MODIFIED
    HTTP_305_USE_PROXY
    HTTP_306_RESERVED
    HTTP_307_TEMPORARY_REDIRECT
    
    # 4）客户端错误 - 4xx
    HTTP_400_BAD_REQUEST
    HTTP_401_UNAUTHORIZED
    HTTP_402_PAYMENT_REQUIRED
    HTTP_403_FORBIDDEN
    HTTP_404_NOT_FOUND
    HTTP_405_METHOD_NOT_ALLOWED
    HTTP_406_NOT_ACCEPTABLE
    HTTP_407_PROXY_AUTHENTICATION_REQUIRED
    HTTP_408_REQUEST_TIMEOUT
    HTTP_409_CONFLICT
    HTTP_410_GONE
    HTTP_411_LENGTH_REQUIRED
    HTTP_412_PRECONDITION_FAILED
    HTTP_413_REQUEST_ENTITY_TOO_LARGE
    HTTP_414_REQUEST_URI_TOO_LONG
    HTTP_415_UNSUPPORTED_MEDIA_TYPE
    HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE
    HTTP_417_EXPECTATION_FAILED
    HTTP_422_UNPROCESSABLE_ENTITY
    HTTP_423_LOCKED
    HTTP_424_FAILED_DEPENDENCY
    HTTP_428_PRECONDITION_REQUIRED
    HTTP_429_TOO_MANY_REQUESTS
    HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE
    HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS
    
    # 5）服务器错误 - 5xx
    HTTP_500_INTERNAL_SERVER_ERROR
    HTTP_501_NOT_IMPLEMENTED
    HTTP_502_BAD_GATEWAY
    HTTP_503_SERVICE_UNAVAILABLE
    HTTP_504_GATEWAY_TIMEOUT
    HTTP_505_HTTP_VERSION_NOT_SUPPORTED
    HTTP_507_INSUFFICIENT_STORAGE
    HTTP_511_NETWORK_AUTHENTICATION_REQUIRED
    ```

#### (2)GenericAPIView[通用视图类]

- 通用视图类主要作用就是把视图中的独特的代码抽取出来，让视图方法中的代码更加通用，方便把通用代码进行简写。
- `GenericAPIView`继承自`APIView`，**主要增加了操作序列化器和数据库查询的方法，作用是为下面Mixin扩展类的执行提供方法支持。通常在使用时，可搭配一个或多个Mixin扩展类。**

##### a.GenericAPIView支持定义的属性

- 列表视图与详情视图通用：
    - queryset列表视图的查询集
    - serializer_class视图使用的序列化器
- 详情页视图使用：
    - lookup_field查询单一数据库对象时使用的条件字段，默认为pk
    - lookup_url_kwarg查询单一数据时URL中的参数关键字名称，默认与look_field相同

##### b.关于序列化器使用的属性与方法

1. `get_serializer_class(self)`

    当出现一个视图类中调用多个序列化器时,那么可以通过条件判断在get_serializer_class方法中通过返回不同的序列化器类名就可以让视图方法执行不同的序列化器对象了。

    返回序列化器类，默认返回`serializer_class`，可以重写

2. `get_serializer(self, *args, **kwargs)`

    返回序列化器对象，主要用来提供给Mixin扩展类使用，如果我们在视图中想要获取序列化器对象，也可以直接调用此方法。

    **注意，该方法在提供序列化器对象的时候，会向序列化器对象的context属性补充三个数据：request、format、view，这三个数据对象可以在定义序列化器时使用。**

    - **request** 当前视图的请求对象
    - **view** 当前请求的类视图对象
    - **format** 当前请求期望返回的数据格式

3. `get_queryset(self)`

    返回视图使用的查询集，主要用来提供给Mixin扩展类使用，是列表视图与详情视图获取数据的基础，默认返回`queryset`属性，可以重写，例如：

    ```python
    def get_queryset(self):
        user = self.request.user
        return user.accounts.all()
    ```

4. `get_object(self)`

    返回详情视图所需的模型类数据对象，主要用来提供给Mixin扩展类使用。

    在试图中可以调用该方法获取详情信息的模型类对象。

    **若详情访问的模型类对象不存在，会返回404。**

    该方法会默认使用APIView提供的check_object_permissions方法检查当前对象是否有权限被访问。

    ```python
    # url(r'^books/(?P<pk>\d+)/$', views.BookDetailView.as_view()),
    class BookDetailView(GenericAPIView):
        queryset = BookInfo.objects.all()
        serializer_class = BookInfoSerializer
    
        def get(self, request, pk):
            book = self.get_object() # get_object()方法根据pk参数查找queryset中的数据对象
            serializer = self.get_serializer(book)
            return Response(serializer.data)
    ```

    其他可以设置的属性

    - **pagination_class** 指明分页控制类
    - **filter_backends** 指明过滤控制后端

##### c.通用视图类的应用

- ```python
    from rest_framework import status
    from rest_framework.generics import GenericAPIView
    from rest_framework.response import Response
    
    from .models import Book
    from .sers import BookSerializer
    
    
    class BookView(GenericAPIView):
        # 这两个属性一定要配置
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request):
            bs = BookSerializer(instance=self.get_queryset(), many=True)
            return Response(bs.data)
    
        def post(self, request):
            serializer = self.get_serializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            return Response(serializer.data)
    ```

### 2、5个视图扩展类（Mixin扩展类）

- 作用：

    - 提供了几种后端视图（对数据资源进行增删改查）处理流程的实现，如果需要编写的视图属于这五种，则视图可以通过继承相应的扩展类来复用代码，减少自己编写的代码量。

    - 这五个扩展类需要搭配GenericAPIView通用视图基类，因为五个扩展类的实现需要调用GenericAPIView提供的序列化器与数据库查询的方法。

#### (1)ListModelMixin

- 列表视图扩展类，提供`list(request, *args, **kwargs)`方法快速实现列表视图，返回200状态码。

    该Mixin的list方法会对数据进行过滤和分页。

- 源码

    ```python
    class ListModelMixin:
        """
        List a queryset.
        """
        def list(self, request, *args, **kwargs):
            queryset = self.filter_queryset(self.get_queryset())
    
            page = self.paginate_queryset(queryset)
            if page is not None:
                serializer = self.get_serializer(page, many=True)
                return self.get_paginated_response(serializer.data)
    
            serializer = self.get_serializer(queryset, many=True)
            return Response(serializer.data)
    ```

- 示例

    ```python
    class BookView(GenericAPIView,ListModelMixin):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request):
            return self.list(request)
    ```

#### (2)CreateModelMixin

- 创建视图扩展类，提供`create(request, *args, **kwargs)`方法快速实现创建资源的视图，成功返回201状态码。

    如果序列化器对前端发送的数据验证失败，返回400错误。

- 源码

    ```python
    class CreateModelMixin:
        """
        Create a model instance.
        """
        def create(self, request, *args, **kwargs):
            serializer = self.get_serializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            self.perform_create(serializer)
            headers = self.get_success_headers(serializer.data)
            return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)
    
        def perform_create(self, serializer):
            serializer.save()
    
        def get_success_headers(self, data):
            try:
                return {'Location': str(data[api_settings.URL_FIELD_NAME])}
            except (TypeError, KeyError):
                return {}
    ```

- 示例

    ```python
    class BookView(GenericAPIView, ListModelMixin, CreateModelMixin):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request):
            return self.list(request)
    
        def post(self, request):
            return self.create(request)
    ```

#### (3)RetrieveModelMixin

- 详情视图扩展类，提供`retrieve(request, *args, **kwargs)`方法，可以快速实现返回一个存在的数据对象。

    如果存在，返回200， 否则返回404。

- 源码

    ```python
    class RetrieveModelMixin:
        """
        Retrieve a model instance.
        """
        def retrieve(self, request, *args, **kwargs):
            instance = self.get_object()
            serializer = self.get_serializer(instance)
            return Response(serializer.data)
    ```

- 示例

    ```python
    class BookDetailView(GenericAPIView,RetrieveModelMixin):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request, pk):
            return self.retrieve(request, pk)
    ```

#### (4)UpdateModelMixin

- 更新视图扩展类，提供`update(request, *args, **kwargs)`方法，可以快速实现更新一个存在的数据对象。

    同时也提供`partial_update(request, *args, **kwargs)`方法，可以实现局部更新。

    成功返回200，序列化器校验数据失败时，返回400错误。

- 源码

    ```python
    class UpdateModelMixin:
        """
        Update a model instance.
        """
        def update(self, request, *args, **kwargs):
            partial = kwargs.pop('partial', False)
            instance = self.get_object()
            serializer = self.get_serializer(instance, data=request.data, partial=partial)
            serializer.is_valid(raise_exception=True)
            self.perform_update(serializer)
    
            if getattr(instance, '_prefetched_objects_cache', None):
                # If 'prefetch_related' has been applied to a queryset, we need to
                # forcibly invalidate the prefetch cache on the instance.
                instance._prefetched_objects_cache = {}
    
            return Response(serializer.data)
    
        def perform_update(self, serializer):
            serializer.save()
    
        def partial_update(self, request, *args, **kwargs):
            kwargs['partial'] = True
            return self.update(request, *args, **kwargs)
    ```

- 示例

    ```python
    class BookDetailView(GenericAPIView, RetrieveModelMixin, UpdateModelMixin):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request, pk):
            return self.retrieve(request, pk)
    
        def put(self, request, pk):
            return self.update(request, pk)
    ```

#### (5)DestroyModelMixin

- 删除视图扩展类，提供`destroy(request, *args, **kwargs)`方法，可以快速实现删除一个存在的数据对象。

    成功返回204，不存在返回404。

- 源码

    ```python
    class DestroyModelMixin:
        """
        Destroy a model instance.
        """
        def destroy(self, request, *args, **kwargs):
            instance = self.get_object()
            self.perform_destroy(instance)
            return Response(status=status.HTTP_204_NO_CONTENT)
    
        def perform_destroy(self, instance):
            instance.delete()
    ```

- 示例

    ```python
    class BookDetailView(GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin):
        queryset = Book.objects.all()
        serializer_class = BookSerializer
    
        def get(self, request, pk):
            return self.retrieve(request, pk)
    
        def put(self, request, pk):
            return self.update(request, pk)
    
        def delete(self, request, pk):
            return self.destroy(request, pk)
    ```

### 3、GenericAPIView的视图子类

#### (1)CreateAPIView

- 提供了post方法，内部调用了create方法

- 继承自： GenericAPIView、CreateModelMixin

#### (2)ListAPIView

- 提供了get方法，内部调用了list方法

- 继承自：GenericAPIView、ListModelMixin

#### (3)RetrieveAPIView

- 提供了get方法，内部调用了retrieve方法

- 继承自: GenericAPIView、RetrieveModelMixin

#### (4)DestoryAPIView

- 提供了delete方法，内部调用了destory方法

- 继承自：GenericAPIView、DestoryModelMixin

#### (5)UpdateAPIView

- 提供了put和patch方法，内部调用了update和partial_update方法

- 继承自：GenericAPIView、UpdateModelMixin

#### (6)ListCreateAPIView

- 提供了get和post方法，内部调用了list和create方法

- 继承自：GenericAPIView、ListModelMixin、CreateModelMixin

#### (7)RetrieveUpdateAPIView

- 提供 get、put、patch方法

继承自： GenericAPIView、RetrieveModelMixin、UpdateModelMixin

#### (8)RetrieveDestoryAPIView

- 提供 get、delete方法

- 继承自：GenericAPIView、RetrieveModelMixin、DestoryModelMixin

#### (9)RetrieveUpdateDestoryAPIView

- 提供 get、put、patch、delete方法

- 继承自：GenericAPIView、RetrieveModelMixin、UpdateModelMixin、DestoryModelMixin

### 4、视图集

#### (1)ViewSet

- 继承自`APIView`与`ViewSetMixin`，作用也与APIView基本类似，提供了身份认证、权限校验、流量管理等。

    **ViewSet主要通过继承ViewSetMixin来实现在调用as_view()时传入字典{“http请求”：“视图方法”}的映射处理工作，如{‘get’:’list’}**在ViewSet中，没有提供任何动作action方法，需要我们自己实现action方法。

    使用视图集ViewSet，可以将一系列视图相关的代码逻辑和相关的http请求动作封装到一个类中：

    - list() 提供一组数据

    - retrieve() 提供单个数据

    - create() 创建数据

    - update() 保存数据

    - destory() 删除数据

    ViewSet视图集类不再限制视图方法名只允许get()、post()等这种情况了，而是实现允许开发者根据自己的需要定义自定义方法名，例如 list() 、create() 等，然后经过路由中使用http和这些视图方法名进行绑定调用。

- 示例

    视图集只在使用as_view()方法的时候，才会将**action**动作与具体请求方式对应上。如：

    ```python
    from django.urls import path, re_path
    
    from vset.views import BookView
    
    urlpatterns = [
        # path("set", views.BookView.as_view({"http请求":"视图方法"})),
        path("books/", BookView.as_view({
            "get": "get_all_book",
            "post": "add_book"
        })),
        re_path("^books/(?P<pk>\d+)$", BookView.as_view({
            "get": "get_one_book",
            "put": "edit_book",
            "delete": "delete",
        })),
    ]
    ```

    ```python
    from rest_framework import serializers
    from rest_framework import status
    from rest_framework.response import Response
    from rest_framework.viewsets import ViewSet
    
    from .models import Book
    
    
    class BookSerializer(serializers.ModelSerializer):
        class Meta:
            model = Book
            fields = "__all__"
    
    
    class BookView(ViewSet):
    
        def get_all_book(self, request):
    
            books = Book.objects.all()
            bs = BookSerializer(instance=books, many=True)
            return Response(bs.data)
    
        def add_book(self, request):
            bs = BookSerializer(data=request.data)
            if bs.is_valid():
                bs.save()
                return Response(bs.data)
            else:
                return Response(bs.errors)
    
        def get_one_book(self, request, pk):
            book = Book.objects.get(pk=pk)
            bs = BookSerializer(instance=book)
            return Response(bs.data)
    
        def edit_book(self, request, pk):
            instance = Book.objects.get(pk=pk)
            bs = BookSerializer(instance=instance, data=request.data)
            if bs.is_valid():
                bs.save()
                return Response(bs.data)
            else:
                return Response(bs.errors)
    
        def delete(self, request, pk):
            Book.objects.get(pk=pk).delete()
            return Response(status=status.HTTP_204_NO_CONTENT)
    ```

#### (2)GenericViewSet

- 继承自GenericAPIView和ViewSetMixin，作用让视图集的视图代码变得更加通用，抽离独特代码作为视图类的属性。

    使用ViewSet通常并不方便，因为list、retrieve、create、update、destory等方法都需要自己编写，而这些方法与前面讲过的Mixin扩展类提供的方法同名，所以我们可以通过继承Mixin扩展类来复用这些方法而无需自己编写。但是Mixin扩展类依赖与`GenericAPIView`，所以还需要继承`GenericAPIView`。

    **GenericViewSet**就帮助我们完成了这样的继承工作，继承自`GenericAPIView`与`ViewSetMixin`，在实现了调用as_view()时传入字典（如`{'get':'list'}`）的映射处理工作的同时，还提供了`GenericAPIView`提供的基础方法，可以直接搭配Mixin扩展类使用。

- 示例

    ```python
    from django.urls import path, re_path
    
    from vset.views import BookView
    
    urlpatterns = [
        # path("set", views.BookView.as_view({"http请求":"视图方法"})),
        path("books/", BookView.as_view({
            "get": "list",
            "post": "create"
        })),
        re_path("^books/(?P<pk>\d+)$", BookView.as_view({
            "get": "retrieve",
            "put": "update",
            "delete": "delete",
        })),
    ]
    ```

    集合我们上面学习的模型扩展类，实现简写操作，视图，代码：

    ```python
    from rest_framework.viewsets import GenericViewSet
    from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, \
        DestroyModelMixin
    
    
    class BookView(GenericViewSet, ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin,
                              DestroyModelMixin):
        queryset = Book.objects
        serializer_class = BookSerializer
    ```

#### (3)ModelViewSet和ReadOnlyModelViewSet

- ModelViewSet继承自`GenericViewSet`，同时包括了ListModelMixin、RetrieveModelMixin、CreateModelMixin、UpdateModelMixin、DestoryModelMixin。

    ReadOnlyModelViewSet承自`GenericViewSet`，同时包括了ListModelMixin、RetrieveModelMixin。

- 示例

    ```python
    from rest_framework.viewsets import ModelViewSet
    
    class BookView(ModelViewSet):
        queryset = Book.objects
        serializer_class = BookSerializer
        # 什么方法都不用写，欸嘿
    ```

## 九、路由-Routers

## 十、其他组件
