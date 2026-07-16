Laravel 为构建 REST API 提供了一个干净的接口。Rest API 实际上是构建灵活且可扩展的 API 的方式。但这是有代价的，因为随着 API 的增长，API 返回的有效负载也会增加。这会导致性能下降，尤其是在应用程序有很多资源的情况下。这就是 GraphQL 的用武之地。

Graphql 是一种查询语言和 API 运行时，由 Facebook 开发。它为传统的 REST API 提供了一种更高效、更灵活的替代方案，允许客户端准确地指定它需要的数据，而不是让服务器返回一组固定的数据。这导致更快、更高效的 API 调用，使其成为现代 web 开发的理想选择。

Shopify 等平台使用 GraphQL 和 REST 构建了 API，允许开发人员选择连接到哪种类型的 API。

在本教程中，我们将探讨使用 GraphQL 的好处，以及如何为您的下一个 Laravel 项目设置 GraphQL。

我们将介绍从安装 GraphQL 包到构建简单的 GraphQLAPI 以及探索身份验证和分页等高级主题的所有内容。

无论是初学者还是经验丰富的 Laravel 开发人员，本教程都将为您提供开始使用 Laravel GraphQL 所需的知识。所以，让我们深入研究吧！

## **GraphQL 的关键概念**

在深入学习教程之前，我们需要学习 GraphQL 的一些关键概念。

### **Schema**

GraphQL 模式定义了 GraphQL  的结构。与数据库 schema 类似，它提供了应该如何查询 GraphQL API 的蓝图。

### **查询(Query)**

类似于数据库查询，GraphQL 查询用于从 API 获取数据。它允许客户端从服务器中请求数据。

### **Mutation**

Mutation 允许客户端发送数据到服务器，可以创建(create)、更新(update)或删除(delete)数据。

### **类型**

Graphql 使用类型系统来定义可以查询的数据的数据类型。类型是我们所知道的默认数据类型(Integer、String、Float 等)。

### **Resolver**

Resolver 是用于从服务器检索数据并将其返回给客户端的函数。它们也可以用于突变，以添加/更新数据库中的数据。

### **参数**

这些是附加数据，可以与查询字段一起传递，以进一步自定义 API 返回的数据。参数的示例包括限制和获取分页结果时的页码。  
**订阅(Subscription)**

订阅允许用户在数据改变时从服务器实时接收数据。

### **批量查询**

Graphql 允许在单个请求中执行多个查询，这样可以减少服务器和客户端之间的往返次数。

## **Laravel 中怎么设置 GraphQL 服务器**

为了在 Laravel 中构建 **GraphQL** API，我们需要安装和配置一些关键的东西。

### **安装 Graphql 包**

本教程中，我使用 [rebing/graphql-laravel](https://github.com/rebing/graphql-laravel) 包来添加 graphql 服务器，其他包诸如  [lighthouse-php 包](https://github.com/nuwave/lighthouse)也可使用。它们都带有 graphql playground，允许我们查询 graphql API。

    composer require rebing/graphql-laravel

### **配置包**

发布配置

    php artisan vendor:publish --provider="Rebing\GraphQL\GraphQLServiceProvider"

该命令会发布 `**config/graphql.php**` 文件，我们可以用它来注册查询、mutation 和 schema。

## **创建一个简单的 GraphQL API.**

在这一部分中，我们将着重于创建查询、schema、mutation 和通过 graphiql Playplace 接口测试 API。我将使用一个涉及一对多关联的 User 和 Articles 的简单示例。

GraphQL API 通常包含单个个端点(/**graphql**)，可以通过 POST 请求访问该端点。该端点负责操作所有数据；无论是添加数据、提取数据还是删除数据。通过 schema、查询和 mutation，这是可能的。让我们开始在 Laravel 中构建一个简单的GraphQL API。

### **创建迁移和种子**

我们可以从创建迁移和种子数据库开始。让我们首先创建 Article 的模型、工厂和迁移文件。

    php artisan make:model Articles -m -f

`-m` 标志生成迁移文件，而 `-f` 标志生成对应的工厂类，用于数据库种子。

    //App/Models/Articles.php
    
    <?php
    
    namespace App\Models;
    
    use App\Models\User;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Factories\HasFactory;
    
    class Articles extends Model
    {
        use HasFactory;
    
        protected $fillable = [
            'title', 'author_id', 'content'
        ];
    
        public function author()
        {
            return $this->belongsTo(User::class, 'author_id');
        }
    }
    

Articles 模型

    <?php
    
    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;
    
    return new class extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('articles', function (Blueprint $table) {
                $table->id();
    
                $table->unsignedBigInteger('author_id')->index();
                $table->string('title');
                $table->longText('content');
                $table->timestamps();
    
                $table->foreign('author_id')->references('id')->on('users')->cascadeOnDelete();
            });
        }
    
        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::dropIfExists('articles');
        }
    };

迁移文件

    <?php
    
    namespace Database\Factories;
    
    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Facades\DB;
    
    /**
     * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\Articles>
     */
    class ArticlesFactory extends Factory
    {
        /**
         * Define the model's default state.
         *
         * @return array<string, mixed>
         */
        public function definition()
        {
            $userIDS = DB::table('users')->pluck('id');
    
            return [
                'title' => $this->faker->words(2, true),
                'content' => $this->faker->sentences(3, true),
                'author_id' => $this->faker->randomElement($userIDS)
            ];
        }
    }
    

Article 工厂

然后进行迁移并生成数据库种子

    php artisan migrate --seed

### **创建类型**

每个系统中的“资源”必需进行类型定义。Graphal 用它来定义查询结构。

要创建类型，你可以运行如下命令

    php artisan make:graphql:type ArticleType

此处，我们可以定义可以在客户端访问的字段。可将其理解为 “API 资源”不过以更为强大。

    //App/GraphQL/Types;
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Types;
    
    use App\Models\Articles;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Type as GraphQLType;
    
    class ArticlesType extends GraphQLType
    {
        protected $attributes = [
            'name' => 'Articles',
            'description' => 'A type definition for the Articles Model',
            'model' => Articles::class
        ];
    
        public function fields(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::id()),
                    'description' => 'The auto incremented Article ID'
                ],
                'title' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'A title of the Article'
                ],
                'content' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The Body of the Article'
                ],
                'author' => [
                    'type' => Type::nonNull(GraphQL::type('User')),
                    'description' => 'The Author who wrote the articles'
                ],
            ];
        }
    }
    

类型定义完后，我们可以在后续的查询及 mutation 中使用它们。

### **定义查询**

既然数据库中有了数据，现在我们可以准备查询了。如前所见，查询用以从数据库中获取数据。

运行如下命令创建查询。

    php artisan make:graphql:query Articles/ArticleQuery

该查询将负责返回单个 **Article**.

我们将使用 `id` 作为参数，将其传递给解析器(resolver)，使之返回单个 Article。

    //App/GraphQL/Queries/Articles/ArticleQuery
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Queries\Articles;
    
    use App\Models\Articles;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Query;
    use Rebing\GraphQL\Support\SelectFields;
    
    class ArticleQuery extends Query
    {
        protected $attributes = [
            'name' => 'article',
            'description' => 'A query for fetching a single article'
        ];
    
        public function type(): Type
        {
            return GraphQL::type('Articles');
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'name' => 'id',
                    'type' => Type::nonNull(Type::id())
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            /** @var SelectFields $fields */
            $fields = $getSelectFields();
            $select = $fields->getSelect();
            $with = $fields->getRelations();
    
            return Articles::select($select)->with($with)->findOrFail($args['id']);
        }
    }
    

从代码中可以看到，我们使用了早先创建的默认类型来结构化查询。

接下来，我将创建一个返回所有/分页(`all/paginated`)文章的查询:

    php artisan make:graphql:query Articles/ArticlesQuery

然后添加如下逻辑，使之返回分页相应：

    //App/GraphQL/Queries/Articles/ArticlesQuery
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Queries\Articles;
    
    
    use App\Models\Articles;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Query;
    use Rebing\GraphQL\Support\SelectFields;
    
    class ArticlesQuery extends Query
    {
        protected $attributes = [
            'name' => 'articles',
            'description' => 'A query to fetch all articles'
        ];
    
        public function type(): Type
        {
            // return Type::listOf(GraphQL::type('Articles')); //return all results
            return GraphQL::paginate('Articles'); //return paginated results
        }
    
        public function args(): array
        {
            return [
                'limit' => [
                    'type' => Type::int()
                ],
                'page' => [
                    'type' => Type::int()
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            /** @var SelectFields $fields */
            $fields = $getSelectFields();
            $select = $fields->getSelect();
            $with = $fields->getRelations();
    
            if (!array_key_exists('limit', $args) || !array_key_exists('page', $args)) {
                $articles = Articles::with($with)->select($select)
                    ->paginate(5, ['*'], 'page', 1);
            } else {
                $articles = Articles::with($with)->select($select)
                    ->paginate($args['limit'], ['*'], 'page', $args['page']);
            }
    
            return $articles;
    
            // return Articles::select($select)->with($with)->get(); This returns all records without pagination
        }
    }

类似于单个文章查询，我们也使用了前面创建的 `ArticleType` 来格式化分页结果。

就这样，我们完成了查询。

### **定义 Mutations**

当我们想在数据库中新建(`create`)、更新(`update`)或删除(`delete`)记录时，**Mutations** 很有用。 

#### **新建文章 Mutation**

该 mutation 将会负责将文章记录添加到数据库。

我们将使用此包提供的另一个命令：

    php artisan make:graphql:mutation Articles/CreateArticleMutation

    //App/GraphQL/Mutations/Articles/CreateArticleMutation
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Mutations\Articles;
    
    use App\Models\Articles;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Mutation;
    
    
    class CreateArticleMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'CreateArticle',
            'description' => 'A mutation for Creating an Article'
        ];
    
        public function type(): Type
        {
            return GraphQL::type('Articles');
        }
    
        public function args(): array
        {
            return [
                'title' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'A title of the Article'
                ],
                'content' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The Body of the Article'
                ],
                'author_id' => [
                    'type' => Type::nonNull(Type::id()),
                    'description' => 'The Author of the Article'
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            return Articles::create($args);
        }
    }
    

#### **更新文章 Mutation**

我们也可以在更新数据库中的文章

    php artisan make:graphql:mutation Articles/UpdateArticleMutation

    //App/GraphQL/Mutations/Articles/UpdateArticleMutation
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Mutations\Articles;
    
    use App\Models\Articles;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Facades\GraphQL;
    use Rebing\GraphQL\Support\Mutation;
    
    
    class UpdateArticleMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'UpdateArticle',
            'description' => 'A mutation for Updating an Article'
        ];
    
        public function type(): Type
        {
            return GraphQL::type('Articles');
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::id()),
                    'description' => 'The auto incremented Article ID'
                ],
                'title' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'A title of the Article'
                ],
                'content' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The Body of the Article'
                ],
                'author_id' => [
                    'type' => Type::nonNull(Type::id()),
                    'description' => 'The Author of the Article'
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
    
            $article = Articles::findOrFail($args['id']);
    
            $article->update($args);
    
            return $article;
        }
    }
    

我们使用 **article id** 作为参数，并传递给解析器(resolver)，使之返回我们可以更新的单篇文章。

#### **删除文章 Mutation**

最后，我们需要准备一个 delete mutation，以完成 CRUD 操作

    php artisan make:graphql:mutation Articles/DeleteArticlesMutation

    //App/GraphQL/Mutations/Articles/DeleteArticleMutation
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Mutations\Articles;
    
    use App\Models\Articles;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Rebing\GraphQL\Support\Mutation;
    
    
    class DeleteArticleMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'DeleteArticle',
            'description' => 'A mutation for deleting an Article'
        ];
    
        public function type(): Type
        {
            return Type::boolean();
        }
    
        public function args(): array
        {
            return [
                'id' => [
                    'type' => Type::nonNull(Type::id()),
                    'description' => 'The auto incremented Article ID'
                ],
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            $article = Articles::findOrFail($args['id']);
    
            return $article->delete();
        }
    }
    

类似于 Update Mutation，我们使用 **article’s id** 来解析文章，然后删除。

删除完记录后，在 REST API 中我们通常返回 204(No Content)状态码。不过，在 GraphQL API 中不可能如此，因此**我们需要返回布尔值**，这样我们才能知道该 mutation 是否成功。对应删除，**响应将返回** `**true**`**。**

既然所有的 mutation 和查询都已创建号，我们可以在 `config/graphql.php` 文件中注册：

    //config/graphql.php
    
    <?php
    
    ...
    
       'schemas' => [
            'default' => [
                'query' => [
                    // Article Queries
                    App\GraphQL\Queries\Articles\ArticleQuery::class,
                    App\GraphQL\Queries\Articles\ArticlesQuery::class,
    
                    // User Queries
                    App\GraphQL\Queries\Users\UsersQuery::class,
                    App\GraphQL\Queries\Users\UserQuery::class,
                ],
                'mutation' => [
                    // Article Mutations
                    App\GraphQL\Mutations\Articles\CreateArticleMutation::class,
                    App\GraphQL\Mutations\Articles\UpdateArticleMutation::class,
                    App\GraphQL\Mutations\Articles\DeleteArticleMutation::class,
    
                    // User Mutations
                    App\GraphQL\Mutations\Users\AddUserMutation::class,
                    App\GraphQL\Mutations\Users\UpdateUserMutation::class,
                    App\GraphQL\Mutations\Users\DeleteUserMutation::class,
                    App\GraphQL\Mutations\Users\LoginMutation::class
                ],
                // The types only available in this schema
                'types' => [
                    App\GraphQL\Types\ArticlesType::class,
                    App\GraphQL\Types\UserType::class,
                ],
    
                // Laravel HTTP middleware
                'middleware' => null,
    
                // Which HTTP methods to support; must be given in UPPERCASE!
                'method' => ['GET', 'POST'],
    
                // An array of middlewares, overrides the global ones
                'execution_middleware' => null,
            ],
        ],
    

> 你可以运行 **artisan optimize** 重写缓存配置

### **使用身份验证**

正如我们所知，身份验证和授权是任何 API 的关键功能，GraphqlAPI 也不例外。我们可以通过 Sanctum 或 Passport 发布访问令牌，并使用它们来授权用户请求。

该包在每个 GraphQL 查询或 Mutation 中都提供了 `authorize()` 方法，我们可以覆盖并添加自定义逻辑来授权请求。

我们可以创建一个 `UserLoginMutation` 来发布令牌。

    php artisan make:graphql:mutation Users/LoginMutation

    //App/GraphQL/Mutations/Users/LoginMutation
    
    <?php
    
    declare(strict_types=1);
    
    namespace App\GraphQL\Mutations\Users;
    
    use App\Models\User;
    use Closure;
    use GraphQL\Type\Definition\ResolveInfo;
    use GraphQL\Type\Definition\Type;
    use Illuminate\Support\Facades\Auth;
    use Rebing\GraphQL\Support\Mutation;
    
    class LoginMutation extends Mutation
    {
        protected $attributes = [
            'name' => 'Login',
            'description' => 'A mutation for login a user'
        ];
    
        public function type(): Type
        {
            return Type::string();
        }
    
        public function args(): array
        {
            return [
                'email' => [
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The email of the user',
                    'rules' => ['max:30']
                ],
                'password' => [
                    'name' => 'password',
                    'type' => Type::nonNull(Type::string()),
                    'description' => 'The password of the user',
                ]
            ];
        }
    
        public function resolve($root, array $args, $context, ResolveInfo $resolveInfo, Closure $getSelectFields)
        {
            $credentials = [
                'email'=>$args['email'],
                'password'=>$args['password']
            ];
            if (!Auth::attempt($credentials)) {
                return 'Invalid login details';
            }
    
            $user = User::where('email', $args['email'])->firstOrFail();
    
            $token = $user->createToken('auth_token')->plainTextToken;
    
            return $token;
        }
    }
    

然后我们可以**在 header 中添加 token**，然后我们在 `authorize` 方法中检测，如果用户被授权则返回记录，而如果用户未授权则返回默认消息。

    //App/GraphQL/Queries/Articles/ArticlesQuery
    <?php
    
    ...
    
    public function authorize($root, array $args, $ctx, ?ResolveInfo $resolveInfo = null, ?Closure $getSelectFields = null): bool
        {
            if (request()->user('sanctum')) {
                return true;
            } else
                return false;
        }
    
        public function getAuthorizationMessage(): string
        {
            return "You are not authorized to perform this action";
        }

这样，我们就在 Laravel 中创建了身份认证 GraphQL API。

**Graphql 与 REST API 有何不同?**

Graphql 不同于 REST API，它**允许客户端从服务器请求特定数据**。这将带来更高性能的应用程序。Graphql 只有一个端点，而 REST API 有多个端点。它的不同之处还在于，它只使用一种请求方法(POST)，而 REST API 使用五种方法(POST、GET、PUT、PATCH 和 DELETE)。

**使用 Graphql 有什么好处?**

Graphql 有助于**提高性能**，因为客户端可以只请求所需的数据。

Graphql 提供了**强类型 schema**，确保客户端和服务器使用正确的数据类型。

Graphql 还有助于**减少过度获取和获取不足**。这是因为数据操作和格式化被转移给客户端，从而允许开发人员检索他们需要的数据。

GraphqlAPI 也是**松耦合**的，这意味着可以在服务器级别进行更改，并且不会给客户端带来破坏性的变更。

## **小结**

与传统的 REST API 相比，Graphql 提供了改进的性能、灵活性和易用性。通过利用 Laravel 和 Graphql 的强大功能，开发人员可以构建可扩展、高效且易于维护的强大 API。

在本文中，我们在 Laravel 中创建了一个简单的 Graphql API，并学习了 Graphql 中的关键概念。

我希望本教程对你有所帮助，并激励你探索 Laravel Graphql 可能性。

感谢阅读。欢迎留言探讨！

