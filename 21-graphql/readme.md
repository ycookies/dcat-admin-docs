# [graphql](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=graphql)

。

# [Laravel GraphQL](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=laravel-graphql)

[![最新稳定版本](https://poser.pugx.org/rebing/graphql-laravel/v/stable)](https://packagist.org/packages/rebing/graphql-laravel) [![执照](https://poser.pugx.org/rebing/graphql-laravel/license)](https://packagist.org/packages/rebing/graphql-laravel) [![测试](https://github.com/rebing/graphql-laravel/workflows/Tests/badge.svg)](https://github.com/rebing/graphql-laravel/actions?query=workflow%3ATests) [![下载](https://img.shields.io/packagist/dt/rebing/graphql-laravel.svg?style=flat-square)](https://packagist.org/packages/rebing/graphql-laravel) [![加入 Slack](https://img.shields.io/badge/slack-join-orange.svg)](https://join.slack.com/t/rebing-graphql/shared_invite/enQtNTE5NjQzNDI5MzQ4LTdhNjk0ZGY1N2U1YjE4MGVlYmM2YTc2YjQ0MmIwODY5MWMwZWIwYmY1MWY4NTZjY2Q5MzdmM2Q3NTEyNDYzZjc)

在 Laravel 10.0+ 上将 Facebook 的 GraphQL 与 PHP 8.1+ 结合使用。它基于[GraphQL 参考实现的 PHP 端口。您可以在](https://github.com/webonyx/graphql-php)[React](https://reactjs.org/)博客上的[GraphQL 简介](https://reactjs.org/blog/2015/05/01/graphql-introduction.html)中找到有关 GraphQL 的更多信息，或者您可以阅读[GraphQL 规范](https://spec.graphql.org/)。[](https://reactjs.org/)[](https://spec.graphql.org/)

-   允许创建**查询**和**变异**作为请求端点
    
-   支持多种架构
    
    -   每个架构查询/变异/类型
    -   每个架构的 HTTP 中间件
    -   每个架构的 GraphQL 执行中间件
-   可以为每个查询/变异定义自定义 GraphQL**解析器中间件**
    
    当使用`SelectFields`该类获得 Eloquent 支持时，可以使用以下附加功能：
    
-   查询返回**类型**，可以有自定义**隐私**设置。
    
-   查询的字段可以选择从数据库中**动态**检索。
    

[与Folklore](https://github.com/folkloreinc/laravel-graphql)的原始软件包相比，它提供了以下功能和改进 ：

-   按操作授权
-   每个字段回调定义其可见性（例如对未经身份验证的用户隐藏）
-   `SelectFields`中可用的抽象`resolve()`，允许高级预加载，从而处理 n+1 问题
-   分页支持
-   服务器端支持[查询批处理](https://www.apollographql.com/blog/batching-client-graphql-queries-a685f5bcd41b/)
-   支持文件上传

## [安装](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=installation)

### [依赖项：](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=dependencies)

-   [Laravel 9.0+](https://github.com/laravel/laravel)
-   [GraphQL PHP](https://github.com/webonyx/graphql-php)

### [安装：](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=installation-1)

通过 Composer 请求包：

    composer require rebing/graphql-laravel

#### [Laravel](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=laravel)

发布配置文件：

    php artisan vendor:publish --provider="Rebing\GraphQL\GraphQLServiceProvider"

查看配置文件：

    config/graphql.php

### [概念](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=concepts)

在深入研究代码之前，最好先熟悉一下 GraphQL 的相关概念。如果您已经熟悉 GraphQL，请跳过此部分。

-   `schema`  
    GraphQL 模式定义了与其相关的所有 `queries`、`mutations` 和 `types`。
-   `queries` and `mutations`   
    你在 GraphQL 请求中调用的“方法”（考虑你的 REST 端点）
-   `types`  
    除了 int 和 string 等原始标量之外，还可以通过自定义类型定义和返回自定义“形状”。它们可以映射到您的数据库模型或基本上任何您想要返回的数据。
-   `resolver`   
    任何时候返回数据，它都会被 `resolved`。通常在`query/mutations`中，它指定检索数据的主要方式（例如使用`SelectFields`或 [数据加载器](https://github.com/overblog/dataloader-php)）

通常，所有查询/变异/类型都是使用`$attributes` 属性和`args()`/`fields()`方法以及`resolve()`方法定义的。

args/fields 再次返回它们支持的每个字段的配置数组。这些字段通常支持这些形状

-   `key` 是字段的名称
    
-   `type`（必需）：此处支持的类型的 GraphQL 说明符
    
    可选键包括：
    
-   `description`：在检查 GraphQL 架构时可用
    
-   `resolve`：覆盖默认字段解析器
    
-   `deprecationReason`：记录某些东西被弃用的原因
    

#### [关于声明字段`nonNull`](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=a-word-on-declaring-a-field-nonnull)

`Type::nonNull()`在任何类型的输入和/或输出字段中恰当地使用是很常见的，实际上是一种很好的做法 。

**类型系统的意图越具体，对消费者来说就越好。**

一些例子

-   如果查询/变异参数需要某个字段，请将其声明为非空
-   如果你知道你的（例如模型）字段永远不会返回 null（例如用户 ID、电子邮件等），则将其声明为不为 null
-   如果返回某个内容的列表，例如标签，其 a) 始终是一个数组（甚至是空的）并且 b) 不得包含`null`值，请像这样声明类型：  
    `Type::nonNull(Type::listOf(Type::nonNull(Type::string())))`

GraphQL 生态系统中存在大量工具，您的类型系统越具体，就越受益。

### [数据加载](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=data-loading)

在 GraphQL 中，加载/检索数据的操作称为“解析”。GraphQL 本身并未**定义**“如何解析”，而是将其留给实现者。

在 Laravel 的环境中，很自然地会假设主要数据来源是 Eloquent。因此，此库提供了一个方便的助手，称为 `SelectFields`，它会尽力预先 [加载关系](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=eager-loading-relationships)并 [避免 n+1 问题](https://www.google.com/search?hl=en&q=n%2B1%20problem)。

请注意，这不是唯一方法，使用称为“数据加载器”的_概念_也很常见 。它们通常利用已解析字段的“延迟”执行，如[graphql-php 解决 n+1 问题](https://github.com/webonyx/graphql-php/blob/master/docs/data-fetching.md#solving-n1-problem)中所述。

要点是，您可以在解析器中使用任何您喜欢的数据源（Eloquent、静态数据、ElasticSearch 结果、缓存等），但您必须注意执行模型，以避免重复获取并对数据执行智能预取。

### [中间件概述](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=middleware-overview)

支持以下中间件概念：

-   HTTP 中间件（例如来自 Laravel）
-   GraphQL 执行中间件
-   GraphQL 解析器中间件

简而言之，中间件_通常_是一个类：

-   使用`handle`方法
-   接收一组固定的参数以及下一个中间件的可调用参数
-   负责调用“下一个”中间件。  
    通常中间件就是这么做的，但也可能决定不这么做，而只是返回
-   可以自由改变传递的参数

#### [HTTP 中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=http-middleware)

任何[与 Laravel 兼容的 HTTP 中间件](https://laravel.com/docs/middleware)都 可以通过配置在全局级别为所有 GraphQL 端点提供 `graphql.route.middleware`，或通过按架构提供 `graphql.schemas.<yourschema>.middleware`。按架构提供的中间件将覆盖全局中间件。

#### [GraphQL 执行中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=graphql-execution-middleware)

GraphQL 请求的处理（以下称为“执行”）经过一组中间件。

`graphql.execution_middleware`它们可以通过在全局级别或每个模式上进行设置`graphql.schemas.<yourschema>.execution_middleware`。

默认情况下，在全局级别提供推荐的中间件集。

注意：GraphQL 请求_本身_的执行也是通过中间件实现的，通常应该最后调用中间件（并且不会调用其他中间件）。如果您对详细信息感兴趣，请参阅 `\Rebing\GraphQL\GraphQL::appendGraphqlExecutionMiddleware`

#### [GraphQL 解析器中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=graphql-resolver-middleware)

应用 HTTP 中间件和执行中间件之后，在调用实际方法**之前，**会针对目标查询/变异执行“解析器中间件” 。`resolve()`

有关更多详细信息，请参阅[解析器中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=resolver-middleware)。

### [架构](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=schemas)

定义 GraphQL 端点需要架构。除了全局中间件之外，您还可以定义多个架构并为其分配不同的**HTTP 中间件**和**执行中间件**。例如：

    'schema' => 'default',
    
    'schemas' => [
        'default' => [
            'query' => [
                ExampleQuery::class,
            ],
            'mutation' => [
                ExampleMutation::class,
            ],
            'types' => [
            
            ],
        ],
        'user' => [
            'query' => [
                App\GraphQL\Queries\ProfileQuery::class
            ],
            'mutation' => [
    
            ],
            'types' => [
            
            ],
            'middleware' => ['auth'],
            // Which HTTP methods to support; must be given in UPPERCASE!
            'method' => ['GET', 'POST'], 
            'execution_middleware' => [
                \Rebing\GraphQL\Support\ExecutionMiddleware\UnusedVariablesMiddleware::class,
            ],
        ],
    ],

某种程度上，架构与配置一起定义了其可访问的路由。根据 的默认配置`prefix = graphql`， _默认_架构可通过 访问`/graphql`。

#### [架构类](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=schema-classes)

您也可以在实现的类中定义模式的配置`ConfigConvertible`。

在您的配置中，您可以引用类的名称，而不是数组。

    'schemas' => [
        'default' => DefaultSchema::class
    ]

    namespace App\GraphQL\Schemas;
    
    use Rebing\GraphQL\Support\Contracts\ConfigConvertible;
    
    class DefaultSchema implements ConfigConvertible
    {
        public function toConfig(): array
        {
            return [
                'query' => [
                    ExampleQuery::class,
                ],
                'mutation' => [
                    ExampleMutation::class,
                ],
                'types' => [
                
                ],
            ];
        }
    }

您可以使用该`php artisan make:graphql:schemaConfig`命令自动创建一个新的架构配置类。

### [创建查询](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=creating-a-query)

首先，您通常会创建一个要从查询中返回的类型。`'model'`仅当指定关系时才需要 Eloquent。

> **注意：**`selectable`如果是非数据库字段或不是关系，则键是必需的

    namespace App\GraphQL\Types;
    
    use App\User;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class UserType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'User',
            'description'   => 'A user',
            // Note: only necessary if you use `SelectFields`
            'model'         => User::class,
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The id of the user',
                    // Use 'alias', if the database column is different from the type name.
                    // This is supported for discrete values as well as relations.
                    // - you can also use `DB::raw()` to solve more complex issues
                    // - or a callback returning the value (string or `DB::raw()` result)
                    'alias' => 'user_id',
                ],
                'email' => [
                    'type' => Type::string(),
                    'description' => 'The email of user',
                    'resolve' => function($root, array $args) {
                        // If you want to resolve the field yourself,
                        // it can be done here
                        return strtolower($root->email);
                    }
                ],
                // Uses the 'getIsMeAttribute' function on our custom User model
                'isMe' => [
                    'type' => Type::boolean(),
                    'description' => 'True, if the queried user is the current user',
                    'selectable' => false, // Does not try to query this from the database
                ]
            ];
        }
    
        // You can also resolve a field by declaring a method in the class
        // with the following format resolve[FIELD_NAME]Field()
        protected function resolveEmailField($root, array $args)
        {
            return strtolower($root->email);
        }
    }

最佳做法是从您的模式开始，`config/graphql.php`并将类型直接添加到您的模式中（例如`default`）：

    'schemas' => [
        'default' => [
            // ...
            
            'types' => [
                App\GraphQL\Types\UserType::class,
            ],

或者，您也可以：

-   在“全局”级别添加类型，例如直接在根配置中添加：
    
        'types' => [
            App\GraphQL\Types\UserType::class,
        ],
    
    在全局级别添加它们允许在不同的模式之间共享它们，但请注意，这可能会使您更难理解在哪里使用哪些类型/字段。
    
-   `GraphQL`或者在服务提供商中使用 Facade 添加类型。
    
        GraphQL::addType(\App\GraphQL\Types\UserType::class);
    

然后您需要定义一个返回此类型（或列表）的查询。您还可以指定可在 resolve 方法中使用的参数。

    namespace App\GraphQL\Queries;
    
    use Closure;
    use App\User;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Query;
    
    class UsersQuery extends Query
    {
        protected $attributes = [
            'name' => 'users',
        ];
    
        public function type(): Type
        {
            return Type::nonNull(Type::listOf(Type::nonNull(GraphQL::type('User'))));
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id', 
                    'type' => Type::string(),
                ],
                'email' => [
                    'name' => 'email', 
                    'type' => Type::string(),
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            if (isset($args['id'])) {
                return User::where('id' , $args['id'])->get();
            }
    
            if (isset($args['email'])) {
                return User::where('email', $args['email'])->get();
            }
    
            return User::all();
        }
    }

将查询添加到`config/graphql.php`配置文件

    'schemas' => [
        'default' => [
            'query' => [
                App\GraphQL\Queries\UsersQuery::class
            ],
            // ...
        ]
    ]

就是这样。您应该能够通过向 URL `/graphql`（或您在配置中选择的任何内容）发出请求来查询 GraphQL。尝试使用以下`query`输入发出 GET 请求

    query FetchUsers {
        users {
            id
            email
        }
    }

例如，如果你使用 homestead：

    http://homestead.app/graphql?query=query+FetchUsers{users{id,email}}

### [创建一个突变](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=creating-a-mutation)

突变与任何其他查询类似。它接受参数并返回特定类型的对象。突变旨在用于**修改**（突变）服务器上的状态的操作（查询不应执行此操作）。

这是传统的抽象，从技术上讲，您可以在查询解析中做任何您想做的事情，包括改变状态。

例如，更新用户密码的变更。首先需要定义变更：

    namespace App\GraphQL\Mutations;
    
    use Closure;
    use App\User;
    use GraphQL;
    use GraphQL\Type\Definition\Type;
    use GraphQL\Type\Definition\ResolveInfo;
    use Rebing\GraphQL\Support\Mutation;
    
    class UpdateUserPasswordMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'updateUserPassword'
        ];
    
        public function type(): Type
        {
            return Type::nonNull(GraphQL::type('User'));
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id', 
                    'type' => Type::nonNull(Type::string()),
                ],
                'password' => [
                    'name' => 'password', 
                    'type' => Type::nonNull(Type::string()),
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            $user = User::find($args['id']);
            if(!$user) {
                return null;
            }
    
            $user->password = bcrypt($args['password']);
            $user->save();
    
            return $user;
        }
    }

正如您在方法中看到的`resolve()`，您使用参数来更新模型并返回它。

然后您应该将变异添加到`config/graphql.php`配置文件中：

    'schemas' => [
        'default' => [
            'mutation' => [
                App\GraphQL\Mutations\UpdateUserPasswordMutation::class,
            ],
            // ...
        ]
    ]

然后，您可以在端点上使用以下查询来执行变更：

    mutation users {
        updateUserPassword(id: "1", password: "newpassword") {
            id
            email
        }
    }

如果你使用 Homestead：

    http://homestead.app/graphql?query=mutation+users{updateUserPassword(id: "1", password: "newpassword"){id,email}}

#### [文件上传](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=file-uploads)

该库使用[https://github.com/laragraph/utils](https://github.com/laragraph/utils)，符合[https://github.com/jaydenseric/graphql-multipart-request-spec](https://github.com/jaydenseric/graphql-multipart-request-spec)的规范。

您必须将`\Rebing\GraphQL\Support\UploadType`第一个添加到您的`config/graphql`架构类型定义中（全局或在您的架构中）：

    'types' => [
        \Rebing\GraphQL\Support\UploadType::class,
    ],

发送请求时请务必遵循以下格式`multipart/form-data`：

> **警告：**当您上传文件时，Laravel 将使用 FormRequest - 这意味着改变请求的中间件不会产生任何影响。

    namespace App\GraphQL\Mutations;
    
    use Closure;
    use GraphQL;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Mutation;
    
    class UserProfilePhotoMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'userProfilePhoto',
        ];
    
        public function type(): Type
        {
            return GraphQL::type('User');
        }
    
        public function args(): array
        {
            return [
                'profilePicture' => [
                    'name' => 'profilePicture',
                    'type' => GraphQL::type('Upload'),
                    'rules' => ['required', 'image', 'max:1500'],
                ],
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            $file = $args['profilePicture'];
    
            // Do something with file here...
        }
    }

[注意：您可以按照此处的](https://www.xkoji.dev/blog/working-with-file-uploads-using-altair-graphql/)说明使用[Altair](https://altair.sirmuel.design/)测试文件上传实现。[](https://www.xkoji.dev/blog/working-with-file-uploads-using-altair-graphql/)

##### [Vue.js 和 Axios 示例](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=vuejs-and-axios-example)

    <template>
      <div class="input-group">
        <div class="custom-file">
          <input type="file" class="custom-file-input" id="uploadFile" ref="uploadFile" @change="handleUploadChange">
          <label class="custom-file-label" for="uploadFile">
            Drop Files Here to upload
          </label>
        </div>
        <div class="input-group-append">
          <button class="btn btn-outline-success" type="button" @click="upload">Upload</button>
        </div>
      </div>
    </template>
    
    <script>
      export default {
        name: 'FileUploadExample',
        data() {
          return {
            file: null,
          };
        },
        methods: {
          handleUploadChange() {
            this.file = this.$refs.uploadFile.files[0];
          },
          async upload() {
            if (!this.file) {
              return;
            }
            // Creating form data object
            let bodyFormData = new FormData();
            bodyFormData.set('operations', JSON.stringify({
                       // Mutation string
                'query': `mutation uploadSingleFile($file: Upload!) {
                            upload_single_file  (attachment: $file)
                          }`,
                'variables': {"attachment": this.file}
            }));
            bodyFormData.set('operationName', null);
            bodyFormData.set('map', JSON.stringify({"file":["variables.file"]}));
            bodyFormData.append('file', this.file);
    
            // Post the request to GraphQL controller
            let res = await axios.post('/graphql', bodyFormData, {
              headers: {
                "Content-Type": "multipart/form-data"
              }
            });
    
            if (res.data.status.code == 200) {
              // On success file upload
              this.file = null;
            }
          }
        }
      }
    </script>
    
    <style scoped>
    </style>

##### [jQuery 或原生 JavaScript](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=jquery-or-vanilla-javascript)

    <input type="file" id="fileUpload">

    // Get the file from input element
    // In jQuery:
    let file = $('#fileUpload').prop('files')[0];
    // Vanilla JS:
    let file = document.getElementById("fileUpload").files[0];
    
    // Create a FormData object
    let bodyFormData = new FormData();
    bodyFormData.set('operations', JSON.stringify({
             // Mutation string
      'query': `mutation uploadSingleFile($file: Upload!) {
                  upload_single_file  (attachment: $file)
                }`,
      'variables': {"attachment": this.file}
    }));
    bodyFormData.set('operationName', null);
    bodyFormData.set('map', JSON.stringify({"file":["variables.file"]}));
    bodyFormData.append('file', this.file);
    
    // Post the request to GraphQL controller via Axios, jQuery.ajax, or vanilla XMLHttpRequest
    let res = await axios.post('/graphql', bodyFormData, {
      headers: {
        "Content-Type": "multipart/form-data"
      }
    });

### [验证](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=validation)

Laravel 的验证支持查询、变异、输入类型和字段参数。

> **注意：**此支持是“糖衣加糖”，旨在提供便利。在某些情况下，它可能存在限制，在这种情况下，您可以在各自的`resolve()`方法中使用常规 Laravel 验证，就像在常规 Laravel 代码中一样。

支持通过以下方式添加验证规则：

-   `'rules'`支持字段配置键
    -   在查询/突变中声明的字段中`function args()`
    -   在声明的字段的输入类型中`function fields()`
    -   `'args'`为字段声明
-   覆盖`\Rebing\GraphQL\Support\Field::rules`任何查询/变异/输入类型
-   `Validator`或者直接在你的`resolve()`方法中使用 Laravel

使用配置键`'rules'`非常方便，因为它与 GraphQL 类型本身在同一个位置声明。但是，使用此方法可能会遇到某些限制（例如使用 进行多字段验证`*`），在这种情况下，您可以覆盖该`rules()`方法。

#### [在每个参数中定义规则的示例](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=example-defining-rules-in-each-argument)

    class UpdateUserEmailMutation extends Mutation
    {
        //...
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id',
                    'type' => Type::string(),
                    'rules' => ['required']
                ],
                'email' => [
                    'name' => 'email',
                    'type' => Type::string(),
                    'rules' => ['required', 'email']
                ]
            ];
        }
    
        //...
    }

#### [`rules()`使用该方法的示例](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=example-using-the-rules-method)

    namespace App\GraphQL\Mutations;
    
    use Closure;
    use App\User;
    use GraphQL;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Mutation;
    
    class UpdateUserEmailMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'updateUserEmail'
        ];
    
        public function type(): Type
        {
            return GraphQL::type('User');
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id', 
                    'type' => Type::string(),
                ],
                'email' => [
                    'name' => 'email', 
                    'type' => Type::string(),
                ]
            ];
        }
    
        protected function rules(array $args = []): array
        {
            return [
                'id' => ['required'],
                'email' => ['required', 'email'],
                'password' => $args['id'] !== 1337 ? ['required'] : [],
            ];
        }
    
        public function resolve($root, array $args)
        {
            $user = User::find($args['id']);
            if (!$user) {
                return null;
            }
    
            $user->email = $args['email'];
            $user->save();
    
            return $user;
        }
    }

#### [直接使用 Laravel 验证器的示例](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=example-using-laravel39s-validator-directly)

`validate()`在下面示例中调用将抛出该库`ValidationException` 默认提供的Laravel：`error_formatter`

    protected function resolve($root, array $args) {
        \Illuminate\Support\Facades\Validator::make($args, [
            'data.*.password' => 'string|nullable|same:data.*.password_confirmation',
        ])->validate();
    }

配置键的格式`'rules'`或方法返回的规则 `rules()`遵循 Laravel 支持的相同约定，例如：

-   `'rules' => 'required|string`  
    或者
-   `'rules' => ['required', 'string']`  
    或者
-   `'rules' => function (…) { … }`  
    ETC。

对于`args()`方法或`'args'`字段定义，字段名称直接用于验证。但是，对于可以嵌套并多次出现的输入类型，字段名称将映射为例如 `data.0.fieldname`。这是在从`rules()`方法返回规则时导入以理解的。

#### [处理验证错误](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=handling-validation-errors)

异常用于在 GraphQL 响应中传达发生验证错误的信息。使用内置支持时， `\Rebing\GraphQL\Error\ValidationError`会抛出异常。在您的自定义代码中或直接使用 Laravel 时`Validator`，Laravel 的内置功能 `\Illuminate\Validation\ValidationException`也受支持。在这两种情况下，GraphQL 响应都会转换为如下所示的错误格式。

为了支持在 GraphQL 错误响应中返回验证错误， `'extensions'`使用了，因为没有适当的等效项。

在客户端，您可以检查`message`给定的错误是否匹配 `'validation'`，您可以期望`extensions.validation`将每个字段映射到其各自错误的键：

    {
      "data": {
        "updateUserEmail": null
      },
      "errors": [
        {
          "message": "validation",
          "extensions": {
            "validation": {
              "email": [
                "The email is invalid."
              ]
            }
          },
          "locations": [
            {
              "line": 1,
              "column": 20
            }
          ]
        }
      ]
    }

您可以通过在配置中提供自己的配置来定制处理方式`error_formatter` ，替换此库中的默认配置。

#### [自定义错误消息](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=customizing-error-messages)

可以通过重写方法来定制返回的验证错误 `validationErrorMessages`。此方法应以与 Laravel 验证中记录的相同方式返回自定义验证消息数组。例如，要检查`email`参数是否与任何现有数据冲突，您可以执行以下操作：

> **注意：**键应采用`field_name`.`validator_type`格式，这样您可以返回每个验证类型的特定错误。

    public function validationErrorMessages(array $args = []): array
    {
        return [
            'name.required' => 'Please enter your full name',
            'name.string' => 'Your name must be a valid string',
            'email.required' => 'Please enter your email address',
            'email.email' => 'Please enter a valid email address',
            'email.exists' => 'Sorry, this email address is already in use',
        ];
    }

#### [自定义属性](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=customizing-attributes)

可以通过重写方法来自定义验证属性 `validationAttributes`。此方法应以与 Laravel 验证中所述相同的方式返回自定义属性数组。

    public function validationAttributes(array $args = []): array
    {
        return [
            'email' => 'email address',
        ];
    }

#### [其他说明](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=misc-notes)

GraphQL 的某些类型声明可能会取消我们的验证或使某些验证变得不必要。一个很好的例子是使用`Type::nonNull()`最终声明一个参数是必需的。在这种情况下，`'rules' => 'required'` 配置可能永远不会被触发，因为 GraphQL 执行引擎已经阻止接受此字段。

或者更清楚地说：如果发生 GraphQL 类型系统违规，那么 Laravel 验证甚至不会执行，因为代码还没有完成。

### [解决方法](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=resolve-method)

解析方法用于查询和变异，并且在这里创建响应。

resolve 方法的前三个参数是硬编码的：

1.  此解析方法所属的对象`$root`（可以是`null`）
2.  传递的参数为`array $args`（可以是空数组）
3.  可以通过实现自定义的“执行中间件”来定制查询特定的 GraphQL 上下文，请参阅 `\Rebing\GraphQL\Support\ExecutionMiddleware\AddAuthUserContextValueMiddleware` 示例。

此后将尝试注入参数，类似于 Laravel 中的控制器方法的工作方式。

您可以输入您需要实例的任何类的提示。

有两个硬编码类依赖于查询的本地数据：

-   `GraphQL\Type\Definition\ResolveInfo`具有对现场解决过程有用的信息。
-   `Rebing\GraphQL\Support\SelectFields`允许预先加载相关的 Eloquent 模型，请参阅[预先加载关系](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=eager-loading-relationships)。

例子：

    namespace App\GraphQL\Queries;
    
    use Closure;
    use App\User;
    use GraphQL;
    use GraphQL\Type\Definition\Type;
    use GraphQL\Type\Definition\ResolveInfo;
    use Rebing\GraphQL\Support\SelectFields;
    use Rebing\GraphQL\Support\Query;
    use SomeClassNamespace\SomeClassThatDoLogging;
    
    class UsersQuery extends Query
    {
        protected $attributes = [
            'name' => 'users',
        ];
    
        public function type(): Type
        {
            return Type::listOf(GraphQL::type('User'));
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id', 
                    'type' => Type::string(),
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $info, SelectFields $fields, SomeClassThatDoLogging $logging)
        {
            $logging->log('fetched user');
    
            $select = $fields->getSelect();
            $with = $fields->getRelations();
    
            $users = User::select($select)->with($with);
    
            return $users->get();
        }
    }

### [解析器中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=resolver-middleware)

这些是**GraphQL 特定的解析器中间件**，仅在概念上与 Laravel 的“HTTP 中间件”相关。主要区别如下：

-   Laravel 的 HTTP 中间件：
    -   在架构/路线级别上工作
    -   与任何常规 Laravel HTTP 中间件兼容
    -   对于架构中的所有查询/变更都是相同的
-   解析器中间件
    -   概念上类似
    -   但适用于查询/变异级别，即每个查询/变异可能不同
    -   技术上与 HTTP 中间件不兼容
    -   采取不同的论点

#### [定义中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=defining-middleware)

要创建新的中间件，请使用`make:graphql:middleware`Artisan 命令

    php artisan make:graphql:middleware ResolvePage

此命令将在您的 app/GraphQL/Middleware 目录中放置一个新的 ResolvePage 类。在这个中间件中，我们将 Paginator 当前页面设置为我们通过以下方式接受的参数`PaginationType`：

    namespace App\GraphQL\Middleware;
    
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use Illuminate\Pagination\Paginator;
    use Rebing\GraphQL\Support\Middleware;
    
    class ResolvePage extends Middleware
    {
        public function handle($root, array $args, $context, ResolveInfo $info, Closure $next)
        {
            Paginator::currentPageResolver(function () use ($args) {
                return $args['pagination']['page'] ?? 1;
            });
    
            return $next($root, $args, $context, $info);
        }
    }

#### [注册中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=registering-middleware)

如果您想将中间件分配给特定的查询/变异，请在`$middleware`查询类的属性中列出中间件类。

    namespace App\GraphQL\Queries;
    
    use App\GraphQL\Middleware;
    use Rebing\GraphQL\Support\Query;
    use Rebing\GraphQL\Support\Query;
    
    class UsersQuery extends Query
    {
        protected $middleware = [
            Middleware\Logstash::class,
            Middleware\ResolvePage::class,
        ];
    }

`$middleware`如果您希望在应用程序的每个 GraphQL 查询/变异期间运行中间件，请在基查询类的属性中列出中间件类。

    namespace App\GraphQL\Queries;
    
    use App\GraphQL\Middleware;
    use Rebing\GraphQL\Support\Query as BaseQuery;
    
    abstract class Query extends BaseQuery
    {
        protected $middleware = [
            Middleware\Logstash::class,
            Middleware\ResolvePage::class,
        ];
    }

或者，您可以覆盖`getMiddleware`以提供您自己的逻辑：

        protected function getMiddleware(): array
        {
            return array_merge([...], $this->middleware);
        }

如果要全局注册中间件，请使用`resolver_middleware_append`以下键`config/graphql.php`：

    return [
        ...
        'resolver_middleware_append' => [YourMiddleware::class],
    ];

您还可以`appendGlobalResolverMiddleware`在任何 ServiceProvider 中使用该方法：

        ...
        public function boot()
        {
            ...
            GraphQL::appendGlobalResolverMiddleware(YourMiddleware::class);
            // Or with new instance
            GraphQL::appendGlobalResolverMiddleware(new YourMiddleware(...));
        }

#### [可终止的中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=terminable-middleware)

有时中间件可能需要在响应发送到浏览器后执行一些工作。如果你在中间件上定义了终止方法，并且你的 Web 服务器正在使用 FastCGI，则在响应发送到浏览器后将自动调用终止方法：

    namespace App\GraphQL\Middleware;
    
    use Countable;
    use GraphQL\Language\Printer;
    use GraphQL\Type\Definition\ResolveInfo;
    use Illuminate\Contracts\Pagination\LengthAwarePaginator;
    use Illuminate\Pagination\AbstractPaginator;
    use Illuminate\Support\Arr;
    use Illuminate\Support\Facades\Config;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Facades\Route;
    use Rebing\GraphQL\Support\Middleware;
    
    class Logstash extends Middleware
    {
        public function terminate($root, array $args, $context, ResolveInfo $info, $result): void
        {
            Log::channel('logstash')->info('', (
                collect([
                    'query' => $info->fieldName,
                    'operation' => $info->operation->name->value ?? null,
                    'type' => $info->operation->operation,
                    'fields' => array_keys(Arr::dot($info->getFieldSelection($depth = PHP_INT_MAX))),
                    'schema' => Arr::first(Route::current()->parameters()) ?? Config::get('graphql.default_schema', 'default'),
                    'vars' => $this->formatVariableDefinitions($info->operation->variableDefinitions),
                ])
                    ->when($result instanceof Countable, function ($metadata) use ($result) {
                        return $metadata->put('count', $result->count());
                    })
                    ->when($result instanceof AbstractPaginator, function ($metadata) use ($result) {
                        return $metadata->put('per_page', $result->perPage());
                    })
                    ->when($result instanceof LengthAwarePaginator, function ($metadata) use ($result) {
                        return $metadata->put('total', $result->total());
                    })
                    ->merge($this->formatArguments($args))
                    ->toArray()
            ));
        }
    
        private function formatArguments(array $args): array
        {
            return collect(Arr::sanitize($args))
                ->mapWithKeys(function ($value, $key) {
                    return ["\${$key}" => $value];
                })
                ->toArray();
        }
    
        private function formatVariableDefinitions(?iterable $variableDefinitions = []): array
        {
            return collect($variableDefinitions)
                ->map(function ($def) {
                    return Printer::doPrint($def);
                })
                ->toArray();
        }
    }

终止方法接收解析器参数和查询结果。

一旦定义了可终止的中间件，就应该将其添加到查询和变异的中间件列表中。

### [授权](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=authorization)

对于类似于 Laravel 的请求（或中间件）功能的授权，我们可以`authorize()`在查询或突变中覆盖该函数。Laravel`'auth'`中间件的一个示例：

    namespace App\GraphQL\Queries;
    
    use Auth;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    
    class UsersQuery extends Query
    {
        public function authorize($root, array $args, $ctx, ResolveInfo $resolveInfo = null, Closure $getSelectFields = null): bool
        {
            // true, if logged in
            return ! Auth::guest();
        }
    
        // ...
    }

或者我们可以使用通过 GraphQL 查询传递的参数：

    namespace App\GraphQL\Queries;
    
    use Auth;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    
    class UsersQuery extends Query
    {
        public function authorize($root, array $args, $ctx, ResolveInfo $resolveInfo = null, Closure $getSelectFields = null): bool
        {
            if (isset($args['id'])) {
                return Auth::id() == $args['id'];
            }
    
            return true;
        }
    
        // ...
    }

您还可以在授权失败时提供自定义错误消息（默认为未授权）：

    namespace App\GraphQL\Queries;
    
    use Auth;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    
    class UsersQuery extends Query
    {
        public function authorize($root, array $args, $ctx, ResolveInfo $resolveInfo = null, Closure $getSelectFields = null): bool
        {
            if (isset($args['id'])) {
                return Auth::id() == $args['id'];
            }
    
            return true;
        }
    
        public function getAuthorizationMessage(): string
        {
            return 'You are not authorized to perform this action';
        }
    
        // ...
    }

### [隐私](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=privacy)

> **注意：**这仅适用于使用该类`SelectFields`查询 Eloquent 模型时！

您可以为每个类型的字段设置自定义隐私属性。如果某个字段不允许，`null`则会返回。例如，如果您希望用户的电子邮件只能由他们自己访问：

    class UserType extends GraphQLType
    {
        // ...
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type'          => Type::nonNull(Type::string()),
                    'description'   => 'The id of the user'
                ],
                'email' => [
                    'type'          => Type::string(),
                    'description'   => 'The email of user',
                    'privacy'       => function(array $args, $ctx): bool {
                        return $args['id'] == Auth::id();
                    }
                ]
            ];
        }
    
        // ...
    
    }

或者您可以创建一个扩展抽象 GraphQL Privacy 类的类：

    use Auth;
    use Rebing\GraphQL\Support\Privacy;
    
    class MePrivacy extends Privacy
    {
        public function validate(array $queryArgs, $queryContext = null): bool
        {
            return $queryArgs['id'] == Auth::id();
        }
    }

    use MePrivacy;
    
    class UserType extends GraphQLType
    {
    
        // ...
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type'          => Type::nonNull(Type::string()),
                    'description'   => 'The id of the user'
                ],
                'email' => [
                    'type'          => Type::string(),
                    'description'   => 'The email of user',
                    'privacy'       => MePrivacy::class,
                ]
            ];
        }
    
        // ...
    
    }

### [查询变量](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=query-variables)

GraphQL 为您提供了在查询中使用变量的可能性，因此您无需“硬编码”值。这样做：

    query FetchUserByID($id: String)
    {
        user(id: $id) {
            id
            email
        }
    }

当您查询 GraphQL 端点时，您可以传递 JSON 编码的`variables`参数。

    http://homestead.app/graphql?query=query+FetchUserByID($id:Int){user(id:$id){id,email}}&variables={"id":123}

### [自定义字段](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=custom-field)

如果您想在多种类型中重复使用某个字段，您也可以将其定义为一个类。

    namespace App\GraphQL\Fields;
    
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Field;
    
    class PictureField extends Field
    {
        protected $attributes = [
            'description'   => 'A picture',
        ];
    
        public function type(): Type
        {
            return Type::string();
        }
    
        public function args(): array
        {
            return [
                'width' => [
                    'type' => Type::int(),
                    'description' => 'The width of the picture'
                ],
                'height' => [
                    'type' => Type::int(),
                    'description' => 'The height of the picture'
                ]
            ];
        }
    
        protected function resolve($root, array $args)
        {
            $width = isset($args['width']) ? $args['width']:100;
            $height = isset($args['height']) ? $args['height']:100;
    
            return 'http://placehold.it/'.$width.'x'.$height;
        }
    }

然后您可以在类型声明中使用它

    namespace App\GraphQL\Types;
    
    use App\GraphQL\Fields\PictureField;
    use App\User;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class UserType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'User',
            'description'   => 'A user',
            'model'         => User::class,
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The id of the user'
                ],
                'email' => [
                    'type' => Type::string(),
                    'description' => 'The email of user'
                ],
                //Instead of passing an array, you pass a class path to your custom field
                'picture' => PictureField::class
            ];
        }
    }

#### [更好的可重复使用字段](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=even-better-reusable-fields)

除了使用类名，您还可以提供 的实际实例`Field`。这不仅允许您重新使用该字段，而且还提供了重新使用解析器的可能性。

假设我们想要一种可以以各种方式输出格式的日期的字段类型。

    namespace App\GraphQL\Fields;
    
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Field;
    
    class FormattableDate extends Field
    {
        protected $attributes = [
            'description' => 'A field that can output a date in all sorts of ways.',
        ];
    
        public function __construct(array $settings = [])
        {
            $this->attributes = \array_merge($this->attributes, $settings);
        }
    
        public function type(): Type
        {
            return Type::string();
        }
    
        public function args(): array
        {
            return [
                'format' => [
                    'type' => Type::string(),
                    'defaultValue' => 'Y-m-d H:i',
                    'description' => 'Defaults to Y-m-d H:i',
                ],
                'relative' => [
                    'type' => Type::boolean(),
                    'defaultValue' => false,
                ],
            ];
        }
    
        protected function resolve($root, array $args): ?string
        {
            $date = $root->{$this->getProperty()};
    
            if (!$date instanceof Carbon) {
                return null;
            }
    
            if ($args['relative']) {
                return $date->diffForHumans();
            }
    
            return $date->format($args['format']);
        }
    
        protected function getProperty(): string
        {
            return $this->attributes['alias'] ?? $this->attributes['name'];
        }
    }

您可以在您的类型中使用该字段，如下所示：

    namespace App\GraphQL\Types;
    
    use App\GraphQL\Fields\FormattableDate;
    use App\User;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class UserType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'User',
            'description'   => 'A user',
            'model'         => User::class,
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The id of the user'
                ],
                'email' => [
                    'type' => Type::string(),
                    'description' => 'The email of user'
                ],
    
                // You can simply supply an instance of the class
                'dateOfBirth' => new FormattableDate,
    
                // Because the constructor of `FormattableDate` accepts our the array of parameters,
                // we can override them very easily.
                // Imagine we want our field to be called `createdAt`, but our database column
                // is called `created_at`:
                'createdAt' => new FormattableDate([
                    'alias' => 'created_at',
                ])
            ];
        }
    }

### [预加载关系](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=eager-loading-relationships)

该类`Rebing\GraphQL\Support\SelectFields`允许预先加载相关的 Eloquent 模型。仅从数据库查询所需的字段。

可以通过解析方法中的**类型提示** 来实例化该类。`SelectFields $selectField`

您还可以通过键入提示来构造类`Closure`。闭包接受一个可选参数，用于分析查询的深度。

您的查询将如下所示：

    namespace App\GraphQL\Queries;
    
    use Closure;
    use App\User;
    use GraphQL;
    use GraphQL\Type\Definition\Type;
    use GraphQL\Type\Definition\ResolveInfo;
    use Rebing\GraphQL\Support\SelectFields;
    use Rebing\GraphQL\Support\Query;
    
    class UsersQuery extends Query
    {
        protected $attributes = [
            'name' => 'users',
        ];
    
        public function type(): Type
        {
            return Type::listOf(GraphQL::type('User'));
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id', 
                    'type' => Type::string(),
                ],
                'email' => [
                    'name' => 'email', 
                    'type' => Type::string(),
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $info, Closure $getSelectFields)
        {
            /** @var SelectFields $fields */
            $fields = $getSelectFields();
            $select = $fields->getSelect();
            $with = $fields->getRelations();
    
            $users = User::select($select)->with($with);
    
            return $users->get();
        }
    }

您的 User 类型可能如下所示。`profile`和`posts` 关系也必须存在于 UserModel 的关系中。如果关系需要加载或验证等某些字段，那么您可以定义一个 `always`属性，该属性将添加给定的属性以供选择。

该属性可以是逗号分隔的字符串或始终包含的属性数组。

    namespace App\GraphQL\Types;
    
    use App\User;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class UserType extends GraphQLType
    {
        /**
         * @var array
         */
        protected $attributes = [
            'name'          => 'User',
            'description'   => 'A user',
            'model'         => User::class,
        ];
    
        /**
        * @return array
        */
        public function fields(): array
        {
            return [
                'uuid' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The uuid of the user'
                ],
                'email' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The email of user'
                ],
                'profile' => [
                    'type' => GraphQL::type('Profile'),
                    'description' => 'The user profile',
                ],
                'posts' => [
                    'type' => Type::listOf(GraphQL::type('Post')),
                    'description' => 'The user posts',
                    // Can also be defined as a string
                    'always' => ['title', 'body'],
                ]
            ];
        }
    }

此时，我们拥有任何模型所期望的配置文件和帖子类型。

    class ProfileType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'Profile',
            'description'   => 'A user profile',
            'model'         => UserProfileModel::class,
        ];
    
        public function fields(): array
        {
            return [
                'name' => [
                    'type' => Type::string(),
                    'description' => 'The name of user'
                ]
            ];
        }
    }

    class PostType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'Post',
            'description'   => 'A post',
            'model'         => PostModel::class,
        ];
    
        public function fields(): array
        {
            return [
                'title' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The title of the post'
                ],
                'body' => [
                    'type' => Type::string(),
                    'description' => 'The body the post'
                ]
            ];
        }
    }

### [类型关系查询](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=type-relationship-query)

> **注意：**这仅适用于使用该类`SelectFields`查询 Eloquent 模型时！

您还可以`query`通过 Eloquent 的查询生成器指定将包含在关系中的内容：

    class UserType extends GraphQLType
    {
    
        // ...
    
        public function fields(): array
        {
            return [
                // ...
    
                // Relation
                'posts' => [
                    'type'          => Type::listOf(GraphQL::type('Post')),
                    'description'   => 'A list of posts written by the user',
                    'args'          => [
                        'date_from' => [
                            'type' => Type::string(),
                        ],
                     ],
                    // $args are the local arguments passed to the relation
                    // $query is the relation builder object
                    // $ctx is the GraphQL context (can be customized by overriding `\Rebing\GraphQL\GraphQLController::queryContext`
                    // The return value should be the query builder or void
                    'query'         => function (array $args, $query, $ctx): void {
                        $query->addSelect('some_column')
                              ->where('posts.created_at', '>', $args['date_from']);
                    }
                ]
            ];
        }
    }

### [分页](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=pagination)

如果查询或变异返回，则将使用分页`PaginationType`。

请注意，除非您使用[解析器中间件](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=defining-middleware)，否则您必须手动提供限制和页面值：

    namespace App\GraphQL\Queries;
    
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Query;
    
    class PostsQuery extends Query
    {
        public function type(): Type
        {
            return GraphQL::paginate('posts');
        }
    
        // ...
    
        public function resolve($root, array $args, $context, ResolveInfo $info, Closure $getSelectFields)
        {
            $fields = $getSelectFields();
    
            return Post::with($fields->getRelations())
                ->select($fields->getSelect())
                ->paginate($args['limit'], ['*'], 'page', $args['page']);
        }
    }

查询`posts(limit:10,page:1){data{id},total,per_page}`可能会返回

    {
        "data": {
            "posts: [
                "data": [
                    {"id": 3},
                    {"id": 5},
                    ...
                ],
                "total": 21,
                "per_page": 10
            ]
        }
    }

请注意，当您请求分页资源时，您需要添加额外的“数据”对象，因为返回的数据会在与返回的分页元数据处于同一级别的数据对象中为您提供分页资源。

[](https://laravel.com/docs/pagination#simple-pagination)如果查询或变异返回，则将使用[简单分页](https://laravel.com/docs/pagination#simple-pagination)`SimplePaginationType`。

    namespace App\GraphQL\Queries;
    
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Query;
    
    class PostsQuery extends Query
    {
        public function type(): Type
        {
            return GraphQL::simplePaginate('posts');
        }
    
        // ...
    
        public function resolve($root, array $args, $context, ResolveInfo $info, Closure $getSelectFields)
        {
            $fields = $getSelectFields();
    
            return Post::with($fields->getRelations())
                ->select($fields->getSelect())
                ->simplePaginate($args['limit'], ['*'], 'page', $args['page']);
        }
    }

### [批处理](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=batching)

批量请求需要通过 POST 请求发送。

您可以通过将多个查询（或变更）分组在一起来一次发送它们。因此，无需创建两个 HTTP 请求：

    POST
    {
        query: "query postsQuery { posts { id, comment, author_id } }"
    }
    
    POST
    {
        query: "mutation storePostMutation($comment: String!) { store_post(comment: $comment) { id } }",
        variables: { "comment": "Hi there!" }
    }

你可以将其作为一个批处理

    POST
    [
        {
            query: "query postsQuery { posts { id, comment, author_id } }"
        },
        {
            query: "mutation storePostMutation($comment: String!) { store_post(comment: $comment) { id } }",
            variables: { "comment": "Hi there!" }
        }
    ]

对于一次发送多个请求的系统，这可以通过将在一定时间间隔内进行的查询分批处理来提高性能。

有一些工具可以帮助你解决这个问题，并可以为你处理批处理，例如[Apollo](https://www.apollographql.com/)

> **关于查询批处理的一个注意事项：**虽然它看起来像是一种“唯一胜利”的情况，但使用批处理可能存在一些缺点：
> 
> -   所有查询/变更都在相同的“进程执行上下文”中执行。  
>     如果您的代码具有在通常的 FastCGI 环境（单个请求/响应）中可能不会显示的副作用，则可能会在此处引起问题。
>     
> -   如果您希望它对包含的每个查询/变异都触发，则“HTTP 中间件”仅对整个批次执行_一次_。这可能与日志记录或速率限制特别相关。另一方面，使用“解析器中间件”，这将按预期工作（尽管解决的问题不同）。  
>       
>     
> -   查询/变异的数量没有限制，  
>     目前没有办法限制这一点。
>     

`batching.enable`可以通过将配置设置为来禁用对批处理的支持`false`。

### [标量类型](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=scalar-types)

GraphQL 带有内置的标量类型，如字符串、整数、布尔值等。可以为特殊用途字段创建自定义标量类型。

一个例子可以是一个链接：而不是使用，`Type::string()`您可以创建一个标量类型`Link`并用 引用它`GraphQL::type('Link')`。

其好处包括：

-   专门的描述，这样您就可以赋予字段更多的含义/目的，而不仅仅是称其为字符串类型
-   显式转换逻辑如下步骤：
    -   从内部逻辑转换为序列化的 GraphQL 输出（`serialize`）
    -   查询/字段输入参数转换 ( `parseLiteral`)
    -   当作为变量传递给查询时（`parseValue`）

这也意味着可以在这些方法中添加验证逻辑，以_确保_传递/接收的值是真实的链接。

标量类型必须实现所有方法；您可以使用快速启动它`artisan make:graphql:scalar <typename>`。然后只需将标量添加到模式中的现有类型即可。

对于更高级的用法，请[参阅有关标量类型的官方文档](https://webonyx.github.io/graphql-php/type-system/scalar-types)。

> **关于性能的注意事项：**请注意标量类型方法中包含的代码。如果您使用包含复杂逻辑来验证字段的自定义标量返回大量字段，则可能会影响您的响应时间。

### [枚举](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=enums)

枚举类型是一种特殊的标量，仅限于一组特定的允许值。点击[此处了解有关枚举的更多信息](https://graphql.org/learn/schema/#enumeration-types)

首先创建一个 Enum 作为 GraphQLType 类的扩展：

    namespace App\GraphQL\Enums;
    
    use Rebing\GraphQL\Support\EnumType;
    
    class EpisodeEnum extends EnumType
    {
        protected $attributes = [
            'name' => 'episode',
            'description' => 'The types of demographic elements',
            'values' => [
                'NEWHOPE' => 'NEWHOPE',
                'EMPIRE' => 'EMPIRE',
                'JEDI' => 'JEDI',
            ],
        ];
    }

> **注意：**在`$attributes['values']`数组中，键是 GraphQL 客户端可以选择的枚举值，而值是您的服务器将接收的内容（枚举将解析为的内容）。

Enum 将像您架构中的任何其他类型一样进行注册`config/graphq.php`：

    'schemas' => [
        'default' => [
            'types' => [
                EpisodeEnum::class,
            ],

然后像这样使用它：

    namespace App\GraphQL\Types;
    
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class TestType extends GraphQLType
    {
        public function fields(): array
        {
            return [
                'episode_type' => [
                    'type' => GraphQL::type('EpisodeEnum')
                ]
            ];
        }
    }

### [工会](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=unions)

联合是一种抽象类型，它只是枚举其他对象类型。联合类型的值实际上是所包含的对象类型之一的值。

如果您需要在同一个查询中返回不相关的类型，它很有用。例如，在执行对多个不同实体的搜索时。

定义 UnionType 的示例：

    namespace App\GraphQL\Unions;
    
    use App\Post;
    use GraphQL;
    use Rebing\GraphQL\Support\UnionType;
    
    class SearchResultUnion extends UnionType
    {
        protected $attributes = [
            'name' => 'searchResult',
        ];
    
        public function types(): array
        {
            return [
                GraphQL::type('Post'),
                GraphQL::type('Episode'),
            ];
        }
    
        public function resolveType($value)
        {
            if ($value instanceof Post) {
                return GraphQL::type('Post');
            } elseif ($value instanceof Episode) {
                return GraphQL::type('Episode');
            }
        }
    }
    

### [接口](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=interfaces)

您可以使用接口来抽象一组字段。[在此处阅读有关接口的更多信息](https://graphql.org/learn/schema/#interfaces)

接口的实现：

    namespace App\GraphQL\Interfaces;
    
    use GraphQL;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\InterfaceType;
    
    class CharacterInterface extends InterfaceType
    {
        protected $attributes = [
            'name' => 'character',
            'description' => 'Character interface.',
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::int()),
                    'description' => 'The id of the character.'
                ],
                'name' => Type::string(),
                'appearsIn' => [
                    'type' => Type::nonNull(Type::listOf(GraphQL::type('Episode'))),
                    'description' => 'A list of episodes in which the character has an appearance.'
                ],
            ];
        }
    
        public function resolveType($root)
        {
            // Use the resolveType to resolve the Type which is implemented trough this interface
            $type = $root['type'];
            if ($type === 'human') {
                return GraphQL::type('Human');
            } elseif  ($type === 'droid') {
                return GraphQL::type('Droid');
            }
        }
    }

实现接口的类型：

    namespace App\GraphQL\Types;
    
    use GraphQL;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    use GraphQL\Type\Definition\Type;
    
    class HumanType extends GraphQLType
    {
        protected $attributes = [
            'name' => 'human',
            'description' => 'A human.'
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::int()),
                    'description' => 'The id of the human.',
                ],
                'name' => Type::string(),
                'appearsIn' => [
                    'type' => Type::nonNull(Type::listOf(GraphQL::type('Episode'))),
                    'description' => 'A list of episodes in which the human has an appearance.'
                ],
                'totalCredits' => [
                    'type' => Type::nonNull(Type::int()),
                    'description' => 'The total amount of credits this human owns.'
                ]
            ];
        }
    
        public function interfaces(): array
        {
            return [
                GraphQL::type('Character')
            ];
        }
    }

#### [支持接口关系自定义查询](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=supporting-custom-queries-on-interface-relations)

如果接口包含与自定义查询的关系，则需要实现`public function types()`返回的数组`GraphQL::type()`，即它可能解析为的所有可能类型（与联合的工作非常相似），以便它可以正确地与一起工作`SelectFields`。

根据前面的代码示例，该方法如下所示：

        public function types(): array
        {
            return[
                GraphQL::type('Human'),
                GraphQL::type('Droid'),
            ];
        }

#### [共享接口字段](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=sharing-interface-fields)

由于您经常需要在具体类型中重复接口的许多字段定义，因此共享接口的定义是有意义的。您可以使用方法访问和重用特定的接口字段`getField(string fieldName): FieldDefinition`。要将所有字段作为数组获取，请使用`getFields(): array`

通过这个，你可以像这样编写`fields`你的类的方法：`HumanType`

    public function fields(): array
    {
        $interface = GraphQL::type('Character');
    
        return [
            $interface->getField('id'),
            $interface->getField('name'),
            $interface->getField('appearsIn'),
    
            'totalCredits' => [
                'type' => Type::nonNull(Type::int()),
                'description' => 'The total amount of credits this human owns.'
            ]
        ];
    }

或者使用如下`getFields`方法：

    public function fields(): array
    {
        $interface = GraphQL::type('Character');
    
        return array_merge($interface->getFields(), [
            'totalCredits' => [
                'type' => Type::nonNull(Type::int()),
                'description' => 'The total amount of credits this human owns.'
            ]
        ]);
    }

### [输入对象](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=input-object)

输入对象类型允许您创建复杂的输入。字段没有参数或解析选项，其类型必须是`InputType`。如果您想验证输入数据，可以添加规则选项。[在此处阅读有关输入对象的更多信息](https://graphql.org/learn/schema/#input-types)

首先创建一个 InputObjectType 作为 GraphQLType 类的扩展：

    namespace App\GraphQL\InputObject;
    
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\InputType;
    
    class ReviewInput extends InputType
    {
        protected $attributes = [
            'name' => 'reviewInput',
            'description' => 'A review with a comment and a score (0 to 5)'
        ];
    
        public function fields(): array
        {
            return [
                'comment' => [
                    'name' => 'comment',
                    'description' => 'A comment (250 max chars)',
                    'type' => Type::string(),
                    // You can define Laravel Validation here
                    'rules' => ['max:250']
                ],
                'score' => [
                    'name' => 'score',
                    'description' => 'A score (0 to 5)',
                    'type' => Type::int(),
                    // You must use 'integer' on rules if you want to validate if the number is inside a range
                    // Otherwise it will validate the number of 'characters' the number can have.
                    'rules' => ['integer', 'min:0', 'max:5']
                ]
            ];
        }
    }

输入对象将像模式中的任何其他类型一样进行注册`config/graphq.php`：

    'schemas' => [
        'default' => [
            'types' => [
                'ReviewInput' => ReviewInput::class
            ],

然后在突变中使用它，例如：

    // app/GraphQL/Type/TestMutation.php
    class TestMutation extends GraphQLType {
    
        public function args(): array
        {
            return [
                'review' => [
                    'type' => GraphQL::type('ReviewInput')
                ]
            ]
        }
    
    }

### [类型修饰符](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=type-modifiers)

可以通过将您选择的类型包装在`Type::nonNull`或`Type::listOf`调用中来应用类型修饰符，或者您可以使用可用的简写语法`GraphQL::type`来构建更复杂的类型。

    GraphQL::type('MyInput!');
    GraphQL::type('[MyInput]');
    GraphQL::type('[MyInput]!');
    GraphQL::type('[MyInput!]!');
    
    GraphQL::type('String!');
    GraphQL::type('[String]');
    GraphQL::type('[String]!');
    GraphQL::type('[String!]!');

### [字段和输入别名](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=field-and-input-alias)

可以为查询和变异参数以及输入对象字段添加别名。

它对于将数据保存到数据库的突变特别有用。

这里您可能希望输入的名称与数据库中的列名不同。

例如，数据库列为`first_name`和`last_name`：

    namespace App\GraphQL\InputObject;
    
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\InputType;
    
    class UserInput extends InputType
    {
        protected $attributes = [
            'name' => 'userInput',
            'description' => 'A user.'
        ];
    
        public function fields(): array
        {
            return [
                'firstName' => [
                    'alias' => 'first_name',
                    'description' => 'The first name of the user',
                    'type' => Type::string(),
                    'rules' => ['max:30']
                ],
                'lastName' => [
                    'alias' => 'last_name',
                    'description' => 'The last name of the user',
                    'type' => Type::string(),
                    'rules' => ['max:30']
                ]
            ];
        }
    }

    namespace App\GraphQL\Mutations;
    
    use Closure;
    use App\User;
    use GraphQL;
    use GraphQL\Type\Definition\Type;
    use GraphQL\Type\Definition\ResolveInfo;
    use Rebing\GraphQL\Support\Mutation;
    
    class UpdateUserMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'updateUser'
        ];
    
        public function type(): Type
        {
            return GraphQL::type('User');
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::string())
                ],
                'input' => [
                    'type' => GraphQL::type('UserInput')
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            $user = User::find($args['id']);
            $user->fill($args['input']));
            $user->save();
    
            return $user;
        }
    }

### [JSON 列](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=json-columns)

在数据库中使用 JSON 列时，字段不会被定义为“关系”，而是带有嵌套数据的简单列。要获取不是数据库关系的嵌套对象，请使用`is_relation`类型中的属性：

    class UserType extends GraphQLType
    {
        // ...
    
        public function fields(): array
        {
            return [
                // ...
    
                // JSON column containing all posts made by this user
                'posts' => [
                    'type'          => Type::listOf(GraphQL::type('Post')),
                    'description'   => 'A list of posts written by the user',
                    // Now this will simply request the "posts" column, and it won't
                    // query for all the underlying columns in the "post" object
                    // The value defaults to true
                    'is_relation' => false
                ]
            ];
        }
    
        // ...
    }

### [字段弃用](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=field-deprecation)

有时您可能希望弃用某个字段，但仍必须保持向后兼容性，直到客户端完全停止使用该字段。您可以使用 [指令](https://www.graphql-tools.com/docs/generate-schema/#descriptions--deprecations)弃用某个字段。如果您添加到字段属性，它将在 GraphQL 文档中被标记为已弃用。您可以使用[Apollo Engine](https://www.apollographql.com/blog/schema-validation-with-apollo-engine-4032456425ba/)`deprecationReason` 在客户端上验证架构。[](https://www.apollographql.com/blog/schema-validation-with-apollo-engine-4032456425ba/)

    namespace App\GraphQL\Types;
    
    use App\User;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class UserType extends GraphQLType
    {
        protected $attributes = [
            'name'          => 'User',
            'description'   => 'A user',
            'model'         => User::class,
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The id of the user',
                ],
                'email' => [
                    'type' => Type::string(),
                    'description' => 'The email of user',
                ],
                'address' => [
                    'type' => Type::string(),
                    'description' => 'The address of user',
                    'deprecationReason' => 'Deprecated due to address field split'
                ],
                'address_line_1' => [
                    'type' => Type::string(),
                    'description' => 'The address line 1 of user',
                ],
                'address_line_2' => [
                    'type' => Type::string(),
                    'description' => 'The address line 2 of user',
                ],
            ];
        }
    }

### [默认字段解析器](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=default-field-resolver)

可以使用配置选项覆盖底层 webonyx/graphql-php 库提供的默认字段解析器`defaultFieldResolver`。

您可以为其定义任何有效的可调用函数（静态类方法、闭包等）：

    'defaultFieldResolver' => [Your\Klass::class, 'staticMethod'],

接收的参数是您的常规“解析”函数签名。

### [宏](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=macros)

如果您想定义一些可以在各种查询、变异和类型中重复使用的帮助程序，您可以在`GraphQL`外观上使用宏方法。

例如，从服务提供商的启动方法：

    namespace App\Providers;
    
    use GraphQL\Type\Definition\Type;
    use Illuminate\Support\ServiceProvider;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            GraphQL::macro('listOf', function (string $name): Type {
                return Type::listOf(GraphQL::type($name));
            });
        }
    }

该`macro`函数接受名称作为其第一个参数，并接受`Closure`作为其第二个参数。

### [自动持久查询支持](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=automatic-persisted-queries-support)

自动持久查询 (APQ) 通过发送较小的请求来提高网络性能，且无需构建时配置。

APQ 默认情况下是禁用的，可以通过配置`apq.enabled=true`或设置环境变量来启用`GRAPHQL_APQ_ENABLE=true`。

持久查询是可以在客户端上生成的 ID 或哈希，而不是整个 GraphQL 查询字符串，然后将其发送到服务器。这种较小的签名可减少带宽利用率并加快客户端加载时间。持久查询尤其适合与 GET 请求配对，从而实现浏览器缓存并与 CDN 集成。

在后台，APQ 使用 Laravel 的缓存来存储/检索查询。它们在存储之前由 GraphQL 解析，因此无需再次解析它们。请参阅那里的各种选项，了解要使用的缓存、前缀、TTL 等。

> 注意：建议在部署后清除缓存以适应模式的变化！

更多信息请参阅：

-   [Apollo - 自动持久查询](https://www.apollographql.com/docs/apollo-server/performance/apq/)
-   [Apollo 链接持久查询 - 协议](https://github.com/apollographql/apollo-link-persisted-queries#protocol)

> 注意：APQ 协议要求将客户端发送的哈希与服务器上计算出的哈希进行比较。如果变异中间件 `TrimStrings`处于活动状态，并且发送的查询包含前导/尾随空格，则这些哈希永远无法匹配，从而导致错误。
> 
> 在这种情况下，请禁用中间件或在散列之前修剪客户端上的查询。

#### [笔记](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=notes)

-   [错误描述与apollo-server](https://github.com/apollographql/apollo-server)一致。

#### [客户端示例](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=client-example)

下面是一个与 Vue/Apollo 的简单集成示例，`createPersistedQueryLink` 自动管理 APQ 流程。

    // [example app.js]
    
    require('./bootstrap');
    
    window.Vue = require('vue');
    
    Vue.component('example-component', require('./components/ExampleComponent.vue').default);
    
    import { ApolloClient } from 'apollo-client';
    import { ApolloLink } from 'apollo-link';
    import { createHttpLink } from 'apollo-link-http';
    import { createPersistedQueryLink } from 'apollo-link-persisted-queries';
    import { InMemoryCache } from 'apollo-cache-inmemory';
    import VueApollo from 'vue-apollo';
    
    const httpLinkWithPersistedQuery = createPersistedQueryLink().concat(createHttpLink({
        uri: '/graphql',
    }));
    
    // Create the apollo client
    const apolloClient = new ApolloClient({
        link: ApolloLink.from([httpLinkWithPersistedQuery]),
        cache: new InMemoryCache(),
        connectToDevTools: true,
    })
    
    const apolloProvider = new VueApollo({
        defaultClient: apolloClient,
    });
    
    Vue.use(VueApollo);
    
    const app = new Vue({
        el: '#app',
        apolloProvider,
    });

    <!-- [example TestComponent.vue] -->
    
    <template>
        <div>
            <p>Test APQ</p>
            <p>-> <span v-if="$apollo.queries.hello.loading">Loading...</span>{{ hello }}</p>
        </div>
    </template>
    
    <script>
        import gql from 'graphql-tag';
        export default {
            apollo: {
                hello: gql`query{hello}`,
            },
            mounted() {
                console.log('Component mounted.')
            }
        }
    </script>

## [其他功能](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=misc-features)

### [检测未使用的变量](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=detecting-unused-variables)

默认情况下，`'variables'`与 GraphQL 查询一起提供的**未被** 使用的查询将被默默忽略。

如果您考虑假设情况，您的查询中有一个可选（可空）参数，并且您为其提供了一个变量参数，但是您输入了错误，那么这可能会被忽视。

例子：

    mutation test($value:ID) {
      someMutation(type:"falbala", optional_id: $value)
    }

提供的变量：

    {
      // Ops! typo in `values`
      "values": "138"
    }

在这种情况下，什么也不会发生并且`optional_id`将被视为未提供。

为了防止出现这种情况，您可以将添加`UnusedVariablesMiddleware`到您的 `execution_middleware`。

## [配置选项](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=configuration-options)

-   `route`  
    保存路由组的所有配置。每个架构都可通过其名称作为专用路由使用。
    -   `prefix`  
        GraphQL 端点的路由前缀，不带前导`/`。  
        默认通过以下方式提供 API`/graphql`
    -   `controller`  
        允许覆盖默认控制器类，以防您想要扩展或替换现有的控制器类（也支持`array`格式）。
    -   `middleware`  
        在没有提供特定于架构的中间件的情况下应用全局 GraphQL 中间件
    -   `group_attributes`  
        附加路由组属性
-   `default_schema`  
    当路由未提供任何架构时，使用的默认架构名称
-   `batching`\\
    -   'enable'  
        是否支持 GraphQL 批处理
-   `error_formatter`  
    此可调用函数将传递 GraphQL 捕获的每个错误的 Error 对象。该方法应返回表示错误的数组。
-   `errors_handler`  
    自定义错误处理。默认处理程序将把异常传递给 laravel 错误处理机制。
-   `security`  
    限制查询复杂性和深度的各种选项，请参阅 [https://webonyx.github.io/graphql-php/security/上的文档](https://webonyx.github.io/graphql-php/security/)
    -   `query_max_complexity`
    -   `query_max_depth`
    -   `disable_introspection`
-   `pagination_type`  
    您可以定义自己的分页类型。
-   `simple_pagination_type`  
    您可以定义自己的简单分页类型。
-   `defaultFieldResolver`  
    覆盖默认字段解析器，请参阅[http://webonyx.github.io/graphql-php/data-fetching/#default-field-resolver](http://webonyx.github.io/graphql-php/data-fetching/#default-field-resolver)
-   `headers`  
    任何将被添加到默认控制器返回的响应的标头
-   `json_encoding_options`  
    从默认控制器返回响应时的​​任何 JSON 编码选项
-   `apq`  
    自动持久查询 (APQ)
    -   `enable`  
        默认情况下它是禁用的。
    -   `cache_driver`  
        使用哪个缓存驱动程序。
    -   `cache_prefix`  
        要使用的缓存前缀。
    -   `cache_ttl`  
        缓存查询多长时间。
-   `detect_unused_variables`  
    如果启用，查询提供但未被使用的变量将引发错误

## [指南](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=guides)

### [从 v1 升级到 v2](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=upgrading-from-v1-to-v2)

虽然版本 2 建立在相同的代码库上并且不会从根本上改变库本身的工作方式，但许多地方都得到了改进，有时会导致不兼容的变化。

-   步骤0：进行备份！
-   重新发布配置文件以了解所有新设置
-   解析器的顺序和参数/类型已经改变：
    -   前：`resolve($root, $array, SelectFields $selectFields, ResolveInfo $info)`
    -   后：`resolve($root, $array, $context, ResolveInfo $info, Closure $getSelectFields)`
    -   如果您现在想要使用 SelectFields，您必须先请求它：`$selectFields = $getSelectFields();`。这样做的主要原因是性能。SelectFields 是一项可选功能，但会消耗资源来遍历 GraphQL 请求 AST 并检查所有类型的配置以应用其魔力。过去它总是被构造，因此消耗资源，即使没有请求也是如此。这已更改为显式形式。
-   许多方法签名声明已发生改变以提高类型安全性，必须进行调整：
    -   方法字段的签名发生了改变：
        -   从`public function fields()`
        -   到`public function fields(): array`
    -   方法 toType 的签名已改变：
        -   从`public function toType()`
        -   到`public function toType(): \GraphQL\Type\Definition\Type`
    -   方法 getFields 的签名已改变：
        -   从`public function getFields()`
        -   到`public function getFields(): array`
    -   方法接口的签名发生了改变：
        -   从`public function interfaces()`
        -   到`public function interfaces(): array`
    -   方法类型的签名已改变：
        -   从`public function types()`
        -   到`public function types(): array`
    -   方法类型的签名已改变：
        -   从`public function type()`
        -   到`public function type(): \GraphQL\Type\Definition\Type`
    -   方法参数的签名已改变：
        -   从`public function args()`
        -   到`public function args(): array`
    -   方法 queryContext 的签名已改变：
        -   从`protected function queryContext($query, $variables, $schema)`
        -   到`protected function queryContext()`
    -   控制器方法查询的签名已改变：
        -   从`function query($query, $variables = [], $opts = [])`
        -   到`function query(string $query, ?array $variables = [], array $opts = []): array`
    -   如果您使用自定义标量类型：
        -   方法 parseLiteral 的签名发生了变化（由于 webonyx 库的升级）：
            -   从`public function parseLiteral($ast)`
            -   到`public function parseLiteral($valueNode, ?array $variables = null)`
-   如果您想使用，`UploadType`现在必须手动将其添加到架构中。方法已消失，您只需像任何其他类型一样通过 引用它即可。`types``::getInstance()``GraphQL::type('Upload')`
-   遵循 Laravel 惯例并使用复数命名空间（例如，新查询放在 中`App\GraphQL\Queries`，不再`App\GraphQL\Query`如此）；相应的`make`命令已调整。这不会破坏任何现有代码，但代码生成将使用新模式。
-   请务必阅读[变更日志](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/CHANGELOG)以了解更多详细信息

### [从民间传说迁移](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=migrating-from-folklore)

[https://github.com/folkloreinc/laravel-graphql](https://github.com/folkloreinc/laravel-graphql)，以前也称为[https://github.com/Folkloreatelier/laravel-graphql](https://github.com/Folkloreatelier/laravel-graphql)

两个代码库非常相似，并且根据您的定制程度，迁移可能会非常快。

注意：此迁移是针对该库的 2.\* 版本编写的。

以下并非万无一失的清单，但应作为指南。如果您不需要执行某些步骤，则不算错误。

**继续操作之前请先备份！**

-   `composer remove folklore/graphql`
-   如果您有自定义的 ServiceProvider 或者手动包含它，请将其删除。重点是现有的 GraphQL 代码不应被触发运行。
-   `composer require rebing/graphql-laravel`
-   发布`config/graphql.php`并调整（前缀、中间件、模式、类型、变异、查询、安全设置）
    -   已删除的设置
        -   `domain`
        -   `resolvers`
    -   `schema`（默认架构）重命名为`default_schema`
    -   `middleware_schema`不存在，它被定义在`schema.<name>.middleware`现在
-   更改命名空间引用：
    -   从`Folklore\`
    -   到`Rebing\`
-   请参阅[从 v1 到 v2 的升级指南，了解所有函数签名的更改](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=upgrading-from-v1-to-v2)
-   该特征`ShouldValidate`不再存在；提供的功能已融入`Field`
-   查询/变异的解析方法的第一个参数现在是`null`（以前它的默认值是一个空数组）

## [性能注意事项](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=performance-considerations)

### [包装类型](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=wrap-types)

您可以包装类型以向查询和突变添加更多信息。与分页工作类似，您可以对要注入的额外数据执行相同操作（[参见测试示例](https://github.com/rebing/graphql-laravel/tree/master/tests/Unit/WithTypeTests)）。例如，在您的查询中：

    public function type(): Type
    {
        return GraphQL::wrapType(
            'PostType',
            'PostMessageType',
            \App\GraphQL\Types\WrapMessagesType::class,
        );
    }
    
    public function resolve($root, array $args)
    {
        return [
            'data' => Post::find($args['post_id']),
            'messages' => new Collection([
                    new SimpleMessage("Congratulations, the post was found"),
                    new SimpleMessage("This post cannot be edited", "warning"),
            ]),
        ];
    }

## [已知限制](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=known-limitations)

### [SelectFields 相关](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=selectfields-related)

-   通过别名解析字段只会解析一次，即使字段具有不同的参数（[问题](https://github.com/rebing/graphql-laravel/issues/604)）。

## [GraphQL 测试客户端](https://jikeadmin.saishiyun.net/books/dcatplus-admin/#/21-graphql/readme?id=graphql-testing-clients)

-   [火营](https://firecamp.io/graphql)
-   [](https://github.com/graphql/graphiql)[通过 laravel-graphiql 集成](https://github.com/mll-lab/laravel-graphiql)[GraphiQL](https://github.com/graphql/graphiql)

