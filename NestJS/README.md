I'll create a comprehensive NestJS interview preparation guide with both simple and technical explanations.

# NestJS - Interview Preparation

## What is NestJS?

### Simple Explanation

NestJS is a framework for building server-side applications with Node.js. Think of it as a structured way to organize your backend code, similar to how Angular organizes frontend code.

It's like having a blueprint for building a house - instead of placing walls randomly, you follow architectural patterns that make your building strong and maintainable.

**Key idea:** NestJS gives you a clear structure with modules, controllers, and services, making your backend organized and scalable.

### Technical Explanation

NestJS is a progressive Node.js framework for building efficient, reliable, and scalable server-side applications. Built with TypeScript (supports JavaScript), it combines elements of Object-Oriented Programming (OOP), Functional Programming (FP), and Functional Reactive Programming (FRP).

**Core characteristics:**

- **TypeScript first:** Full TypeScript support with type safety
- **Modular architecture:** Organized into modules for separation of concerns
- **Dependency Injection:** Built-in IoC (Inversion of Control) container
- **Decorator-based:** Uses TypeScript decorators extensively
- **Platform agnostic:** Works with Express (default) or Fastify
- **Opinionated structure:** Clear architectural patterns
- **Enterprise-ready:** Built for large-scale applications

**Architecture layers:**

```

┌─────────────────────────────────┐
│ Controllers │ (Handle HTTP requests)
├─────────────────────────────────┤
│ Services/Providers │ (Business logic)
├─────────────────────────────────┤
│ Repositories │ (Data access)
├─────────────────────────────────┤
│ Database │ (Data storage)
└─────────────────────────────────┘

```

**Influenced by:**

- Angular (structure, decorators)
- Spring (Java framework)
- ASP.NET Core

---

## Modules

### Simple Explanation

Modules are like folders that group related code together. If you're building an e-commerce app, you might have a UsersModule, ProductsModule, and OrdersModule - each containing everything related to that feature.

**Example:** A UsersModule contains:

- User controller (handles requests)
- User service (business logic)
- User entity (data structure)
- User repository (database operations)

### Technical Explanation

Modules are classes decorated with `@Module()` decorator. They organize application structure and enable feature encapsulation.

#### Basic Module

```typescript
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [], // Other modules this module depends on
  controllers: [UsersController], // Controllers in this module
  providers: [UsersService], // Services/providers in this module
  exports: [UsersService], // Make service available to other modules
})
export class UsersModule {}
```

#### Root Module (AppModule)

```typescript
import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';
import { ProductsModule } from './products/products.module';
import { AuthModule } from './auth/auth.module';

@Module({
  imports: [UsersModule, ProductsModule, AuthModule],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

#### Feature Module

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';

@Module({
  imports: [
    TypeOrmModule.forFeature([User]), // Import User entity for this module
  ],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // Export to make available to other modules
})
export class UsersModule {}
```

#### Shared Module

```typescript
// common.module.ts
import { Module, Global } from '@nestjs/common';
import { ConfigService } from './config.service';
import { LoggerService } from './logger.service';

@Global() // Makes module globally available (no need to import)
@Module({
  providers: [ConfigService, LoggerService],
  exports: [ConfigService, LoggerService],
})
export class CommonModule {}
```

#### Dynamic Module

```typescript
// database.module.ts
import { Module, DynamicModule } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({})
export class DatabaseModule {
  static forRoot(options: any): DynamicModule {
    return {
      module: DatabaseModule,
      imports: [
        TypeOrmModule.forRoot({
          type: 'postgres',
          host: options.host,
          port: options.port,
          username: options.username,
          password: options.password,
          database: options.database,
          autoLoadEntities: true,
          synchronize: options.synchronize,
        }),
      ],
      providers: [],
      exports: [],
    };
  }
}

// Usage in AppModule
@Module({
  imports: [
    DatabaseModule.forRoot({
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

---

## Controllers

### Simple Explanation

Controllers handle incoming HTTP requests and return responses. They're like receptionists at a hotel - they receive requests from customers and direct them to the right service.

**Example:** When someone visits `/users`, the UsersController receives the request and asks UsersService to get the user data.

### Technical Explanation

Controllers are responsible for handling incoming requests and returning responses to the client. They're classes decorated with `@Controller()` decorator.

#### Basic Controller

```typescript
import {
  Controller,
  Get,
  Post,
  Put,
  Delete,
  Param,
  Body,
  Query,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Controller('users') // Route prefix: /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // GET /users
  @Get()
  async findAll(
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10,
  ) {
    return this.usersService.findAll(page, limit);
  }

  // GET /users/:id
  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  // POST /users
  @Post()
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // PUT /users/:id
  @Put(':id')
  async update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(id, updateUserDto);
  }

  // DELETE /users/:id
  @Delete(':id')
  async remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

#### HTTP Decorators

```typescript
@Controller('products')
export class ProductsController {
  // GET /products
  @Get()
  findAll() {}

  // POST /products
  @Post()
  create() {}

  // PUT /products/:id
  @Put(':id')
  update() {}

  // PATCH /products/:id
  @Patch(':id')
  partialUpdate() {}

  // DELETE /products/:id
  @Delete(':id')
  remove() {}

  // Custom HTTP method
  @HttpCode(204)
  @Delete(':id')
  removeNoContent() {}
}
```

#### Request Object

```typescript
import { Controller, Get, Req, Res, Headers, Ip } from '@nestjs/common';
import { Request, Response } from 'express';

@Controller('example')
export class ExampleController {
  // Access full request object
  @Get('request')
  getRequest(@Req() request: Request) {
    return {
      headers: request.headers,
      query: request.query,
      params: request.params,
      body: request.body,
    };
  }

  // Access specific headers
  @Get('headers')
  getHeaders(@Headers('user-agent') userAgent: string) {
    return { userAgent };
  }

  // Access client IP
  @Get('ip')
  getIp(@Ip() ip: string) {
    return { ip };
  }

  // Manual response handling
  @Get('manual')
  manual(@Res() response: Response) {
    return response.status(200).json({ message: 'Manual response' });
  }
}
```

#### Route Parameters

```typescript
@Controller('users')
export class UsersController {
  // Single parameter: /users/123
  @Get(':id')
  findOne(@Param('id') id: string) {
    return `User #${id}`;
  }

  // Multiple parameters: /users/123/posts/456
  @Get(':userId/posts/:postId')
  findUserPost(
    @Param('userId') userId: string,
    @Param('postId') postId: string,
  ) {
    return `User #${userId}, Post #${postId}`;
  }

  // All params at once
  @Get(':id')
  getAll(@Param() params: any) {
    return params.id;
  }
}
```

#### Query Parameters

```typescript
@Controller('products')
export class ProductsController {
  // /products?page=1&limit=10&sort=name
  @Get()
  findAll(
    @Query('page') page: number = 1,
    @Query('limit') limit: number = 10,
    @Query('sort') sort: string = 'createdAt',
  ) {
    return this.productsService.findAll({ page, limit, sort });
  }

  // All query params at once
  @Get('search')
  search(@Query() query: any) {
    return this.productsService.search(query);
  }
}
```

#### Request Body

```typescript
@Controller('users')
export class UsersController {
  // POST /users with JSON body
  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // Access specific body properties
  @Post('register')
  register(@Body('email') email: string, @Body('password') password: string) {
    return this.authService.register(email, password);
  }
}
```

#### Async/Await

```typescript
@Controller('users')
export class UsersController {
  constructor(private usersService: UsersService) {}

  // Async/await
  @Get()
  async findAll(): Promise<User[]> {
    return await this.usersService.findAll();
  }

  // Promise
  @Get(':id')
  findOne(@Param('id') id: string): Promise<User> {
    return this.usersService.findOne(id);
  }

  // Observable (RxJS)
  @Get('stream')
  findStream(): Observable<User[]> {
    return this.usersService.findStream();
  }
}
```

#### Status Codes and Headers

```typescript
import { Controller, Post, HttpCode, Header } from '@nestjs/common';

@Controller('auth')
export class AuthController {
  // Custom status code
  @Post('login')
  @HttpCode(200) // Default for POST is 201
  login() {}

  // Custom headers
  @Get('data')
  @Header('Cache-Control', 'no-store')
  @Header('X-Custom-Header', 'custom-value')
  getData() {}
}
```

---

## Services (Providers)

### Simple Explanation

Services contain your business logic - the actual work your application does. Controllers receive requests and call services to do the work.

**Example:** UsersController receives a request to create a user. It calls UsersService, which validates the data, hashes the password, saves to database, and sends a welcome email.

### Technical Explanation

Services (Providers) are classes that can be injected as dependencies. They contain business logic and are decorated with `@Injectable()`.

#### Basic Service

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './entities/user.entity';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  async findAll(page: number = 1, limit: number = 10): Promise<User[]> {
    return this.usersRepository.find({
      skip: (page - 1) * limit,
      take: limit,
    });
  }

  async findOne(id: string): Promise<User> {
    const user = await this.usersRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.usersRepository.findOne({ where: { email } });
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(createUserDto);
    return this.usersRepository.save(user);
  }

  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id);
    Object.assign(user, updateUserDto);
    return this.usersRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await this.usersRepository.remove(user);
  }
}
```

#### Service with Dependencies

```typescript
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { EmailService } from '../email/email.service';
import { LoggerService } from '../logger/logger.service';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private emailService: EmailService,
    private loggerService: LoggerService,
  ) {}

  async register(email: string, password: string) {
    // Use injected services
    const user = await this.usersService.create({ email, password });
    await this.emailService.sendWelcome(user.email);
    this.loggerService.log(`New user registered: ${user.email}`);
    return user;
  }
}
```

#### Custom Provider Scopes

```typescript
import { Injectable, Scope } from '@nestjs/common';

// Default: SINGLETON - one instance shared across entire application
@Injectable({ scope: Scope.DEFAULT })
export class SingletonService {}

// REQUEST - new instance for each request
@Injectable({ scope: Scope.REQUEST })
export class RequestScopedService {
  constructor(@Inject(REQUEST) private request: Request) {}
}

// TRANSIENT - new instance every time it's injected
@Injectable({ scope: Scope.TRANSIENT })
export class TransientService {}
```

#### Factory Provider

```typescript
// Define factory
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (configService: ConfigService) => {
    const options = configService.get('database');
    return createConnection(options);
  },
  inject: [ConfigService],
};

// Module
@Module({
  providers: [ConfigService, connectionFactory],
  exports: ['CONNECTION'],
})
export class DatabaseModule {}

// Usage
@Injectable()
export class UsersService {
  constructor(@Inject('CONNECTION') private connection: any) {}
}
```

---

## Dependency Injection

### Simple Explanation

Dependency Injection (DI) is like ordering at a restaurant - you don't make your own food, someone brings it to you. Instead of creating dependencies yourself, NestJS provides them to you.

**Example:** UsersController needs UsersService. Instead of `new UsersService()`, NestJS automatically provides it through the constructor.

### Technical Explanation

NestJS uses an Inversion of Control (IoC) container that manages dependencies and their lifecycles.

#### Constructor Injection

```typescript
// Service to be injected
@Injectable()
export class UsersService {
  findAll() {
    return ['user1', 'user2'];
  }
}

// Controller receiving injection
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}

// Module wiring
@Module({
  controllers: [UsersController],
  providers: [UsersService], // Register provider
})
export class UsersModule {}
```

#### Property Injection (not recommended)

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class UsersService {
  @Inject(LoggerService)
  private logger: LoggerService;

  findAll() {
    this.logger.log('Finding all users');
    return [];
  }
}
```

#### Custom Providers

```typescript
// 1. Class provider
@Module({
  providers: [
    UsersService, // Shorthand for below
    {
      provide: UsersService,
      useClass: UsersService
    }
  ]
})

// 2. Value provider
@Module({
  providers: [
    {
      provide: 'CONFIG',
      useValue: {
        apiKey: 'abc123',
        apiUrl: 'https://api.example.com'
      }
    }
  ]
})

// Usage
@Injectable()
export class ApiService {
  constructor(@Inject('CONFIG') private config: any) {}
}

// 3. Factory provider
@Module({
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const config = configService.get('database');
        return await createConnection(config);
      },
      inject: [ConfigService]
    }
  ]
})

// 4. Alias provider
@Module({
  providers: [
    UsersService,
    {
      provide: 'USERS_SERVICE',
      useExisting: UsersService
    }
  ]
})
```

#### Optional Dependencies

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpOptions: any) {
    // httpOptions might be undefined
    console.log(this.httpOptions || 'No options provided');
  }
}
```

---

## DTOs (Data Transfer Objects)

### Simple Explanation

DTOs define the shape of data coming into your application. They're like forms with fields - they specify what data is expected and validate it.

**Example:** CreateUserDto specifies that creating a user requires an email (must be valid email format) and password (minimum 8 characters).

### Technical Explanation

DTOs are objects that define how data is sent over the network. They're used for validation and type safety.

#### Basic DTO

```typescript
// create-user.dto.ts
export class CreateUserDto {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
}

// Controller usage
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}
```

#### DTO with Validation

```typescript
import {
  IsEmail,
  IsString,
  MinLength,
  MaxLength,
  IsOptional,
} from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(50)
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  lastName: string;

  @IsOptional()
  @IsString()
  phoneNumber?: string;
}

// Enable validation in main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
```

#### Advanced Validation

```typescript
import {
  IsEmail,
  IsString,
  MinLength,
  MaxLength,
  Matches,
  IsInt,
  Min,
  Max,
  IsEnum,
  IsDate,
  IsArray,
  ValidateNested,
  IsOptional,
} from 'class-validator';
import { Type } from 'class-transformer';

enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest',
}

class AddressDto {
  @IsString()
  street: string;

  @IsString()
  city: string;

  @IsString()
  @Matches(/^\d{5}$/, { message: 'ZIP code must be 5 digits' })
  zipCode: string;
}

export class CreateUserDto {
  @IsEmail({}, { message: 'Please provide a valid email' })
  email: string;

  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(50)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain uppercase, lowercase, and number',
  })
  password: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  lastName: string;

  @IsInt()
  @Min(18, { message: 'User must be at least 18 years old' })
  @Max(120)
  age: number;

  @IsEnum(UserRole)
  role: UserRole;

  @IsDate()
  @Type(() => Date)
  birthDate: Date;

  @IsArray()
  @IsString({ each: true })
  @IsOptional()
  tags?: string[];

  @ValidateNested()
  @Type(() => AddressDto)
  @IsOptional()
  address?: AddressDto;
}
```

#### Update DTO (Partial)

```typescript
import { PartialType } from '@nestjs/mapped-types';

// Makes all properties optional
export class UpdateUserDto extends PartialType(CreateUserDto) {}

// Equivalent to:
export class UpdateUserDto {
  @IsEmail()
  @IsOptional()
  email?: string;

  @IsString()
  @MinLength(8)
  @IsOptional()
  password?: string;

  // ... all other fields optional
}
```

#### DTO Transformation

```typescript
import { Transform } from 'class-transformer';

export class CreateProductDto {
  @IsString()
  name: string;

  @IsNumber()
  @Transform(({ value }) => parseFloat(value))
  price: number;

  @IsString()
  @Transform(({ value }) => value.toLowerCase())
  category: string;

  @IsArray()
  @Transform(({ value }) => value.split(','))
  tags: string[];
}
```

#### Custom Validation

```typescript
import {
  registerDecorator,
  ValidationOptions,
  ValidationArguments,
} from 'class-validator';

// Custom decorator
export function IsStrongPassword(validationOptions?: ValidationOptions) {
  return function (object: Object, propertyName: string) {
    registerDecorator({
      name: 'isStrongPassword',
      target: object.constructor,
      propertyName: propertyName,
      options: validationOptions,
      validator: {
        validate(value: any, args: ValidationArguments) {
          const hasUpperCase = /[A-Z]/.test(value);
          const hasLowerCase = /[a-z]/.test(value);
          const hasNumber = /\d/.test(value);
          const hasSpecialChar = /[!@#$%^&*]/.test(value);
          return hasUpperCase && hasLowerCase && hasNumber && hasSpecialChar;
        },
        defaultMessage(args: ValidationArguments) {
          return 'Password must contain uppercase, lowercase, number, and special character';
        },
      },
    });
  };
}

// Usage
export class CreateUserDto {
  @IsStrongPassword()
  password: string;
}
```

---

## Middleware

### Simple Explanation

Middleware are functions that run before your route handlers. They can modify the request, log information, check authentication, etc.

**Example:** A logger middleware that logs every request before it reaches the controller.

### Technical Explanation

Middleware is a function called before the route handler. It has access to request and response objects.

#### Functional Middleware

```typescript
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.path}`);
    next(); // Must call next() to pass control to next middleware
  }
}

// Apply in module
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';

@Module({
  // ...
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*'); // Apply to all routes
  }
}
```

#### Route-specific Middleware

```typescript
@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('users'); // Only /users routes

    consumer
      .apply(AuthMiddleware)
      .exclude(
        { path: 'auth/login', method: RequestMethod.POST },
        { path: 'auth/register', method: RequestMethod.POST },
      )
      .forRoutes({ path: '*', method: RequestMethod.ALL });
  }
}
```

#### Multiple Middleware

```typescript
consumer
  .apply(LoggerMiddleware, AuthMiddleware, CorsMiddleware)
  .forRoutes(UsersController);
```

#### Functional Middleware (simpler syntax)

```typescript
// logger.middleware.ts
export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request: ${req.method} ${req.url}`);
  next();
}

// Apply in module
consumer.apply(logger).forRoutes('*');
```

#### Request-scoped Middleware Example

```typescript
@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private authService: AuthService) {}

  async use(req: Request, res: Response, next: NextFunction) {
    const token = req.headers.authorization?.replace('Bearer ', '');

    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const user = await this.authService.verifyToken(token);
      req['user'] = user; // Attach user to request
      next();
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

---

## Guards

### Simple Explanation

Guards determine whether a request should be handled by the route handler. They're like security guards - they check if you're allowed to enter before letting you in.

**Example:** An authentication guard that checks if the user is logged in before allowing access to protected routes.

### Technical Explanation

Guards are classes that implement the `CanActivate` interface. They return a boolean (or Promise/Observable of boolean) indicating whether the request should proceed.

#### Basic Guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return this.validateRequest(request);
  }

  private validateRequest(request: any): boolean {
    // Check if user is authenticated
    return !!request.user;
  }
}

// Apply to route
@Controller('users')
export class UsersController {
  @UseGuards(AuthGuard)
  @Get('profile')
  getProfile() {
    return 'This is protected';
  }
}

// Apply to entire controller
@Controller('users')
@UseGuards(AuthGuard)
export class UsersController {}

// Apply globally
app.useGlobalGuards(new AuthGuard());
```

#### JWT Authentication Guard

```typescript
import {
  Injectable,
  ExecutionContext,
  UnauthorizedException,
} from '@nestjs/common';
import { AuthGuard as PassportAuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends PassportAuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }

  handleRequest(err, user, info) {
    if (err || !user) {
      throw err || new UnauthorizedException('Invalid or expired token');
    }
    return user;
  }
}

// Usage
@Controller('users')
export class UsersController {
  @UseGuards(JwtAuthGuard)
  @Get('me')
  getProfile(@Request() req) {
    return req.user;
  }
}
```

#### Roles Guard

```typescript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

// Custom decorator
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );

    if (!requiredRoles) {
      return true; // No roles required
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}

// Usage
@Controller('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
export class AdminController {
  @Roles('admin')
  @Get('users')
  getAllUsers() {
    return 'Only admins can see this';
  }

  @Roles('admin', 'moderator')
  @Get('posts')
  getAllPosts() {
    return 'Admins and moderators can see this';
  }
}
```

#### Custom Guard with Injectable Dependencies

```typescript
@Injectable()
export class ThrottlerGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private rateLimitService: RateLimitService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const ip = request.ip;

    const isAllowed = await this.rateLimitService.checkLimit(ip);

    if (!isAllowed) {
      throw new TooManyRequestsException('Rate limit exceeded');
    }

    return true;
  }
}
```

---

## Interceptors

### Simple Explanation

Interceptors can transform the response before sending it to the client, or add extra logic before/after method execution. They're like wrappers around your route handlers.

**Example:** An interceptor that transforms all responses to include a timestamp and wraps data in a standard format.

### Technical Explanation

Interceptors bind extra logic before/after method execution, transform results, transform exceptions, extend basic function behavior, or completely override a function.

#### Basic Interceptor

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, any> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => ({
        data,
        timestamp: new Date().toISOString(),
        path: context.switchToHttp().getRequest().url
      }))
    );
  }
}

// Apply to route
@UseInterceptors(TransformInterceptor)
@Get()
findAll() {
  return ['item1', 'item2'];
}

// Response:
// {
//   data: ['item1', 'item2'],
//   timestamp: '2024-01-15T10:30:00.000Z',
//   path: '/items'
// }
```

#### Logging Interceptor

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const now = Date.now();

    this.logger.log(`Incoming Request: ${method} ${url}`);

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const delay = Date.now() - now;
        this.logger.log(
          `Outgoing Response: ${method} ${url} ${response.statusCode} - ${delay}ms`,
        );
      }),
    );
  }
}
```

#### Timeout Interceptor

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  RequestTimeoutException,
} from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // 5 seconds
      catchError((err) => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutException());
        }
        return throwError(() => err);
      }),
    );
  }
}
```

#### Cache Interceptor

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  private cache = new Map<string, any>();

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const key = `${request.method}-${request.url}`;

    // Check cache
    if (this.cache.has(key)) {
      console.log('Returning from cache');
      return of(this.cache.get(key));
    }

    // Not in cache, proceed with request
    return next.handle().pipe(
      tap((response) => {
        console.log('Storing in cache');
        this.cache.set(key, response);
        // Could add expiration logic here
      }),
    );
  }
}
```

#### Error Handling Interceptor

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  HttpException,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      catchError((err) => {
        if (err instanceof HttpException) {
          return throwError(() => err);
        }

        // Transform unknown errors
        return throwError(
          () =>
            new HttpException(
              {
                status: 500,
                error: 'Internal server error',
                message: err.message,
              },
              500,
            ),
        );
      }),
    );
  }
}
```

---

## Pipes

### Simple Explanation

Pipes transform or validate input data before it reaches your route handler. They're like filters - data goes in one end, gets processed, and comes out the other end.

**Example:** A pipe that converts a string ID from the URL to a number, or validates that the request body matches the expected format.

### Technical Explanation

Pipes are classes that implement the `PipeTransform` interface. They're used for transformation and validation.

#### Built-in Pipes

```typescript
import {
  Controller,
  Get,
  Post,
  Param,
  Body,
  Query,
  ParseIntPipe,
  ParseBoolPipe,
  ParseArrayPipe,
  ParseUUIDPipe,
  DefaultValuePipe,
  ValidationPipe,
} from '@nestjs/common';

@Controller('users')
export class UsersController {
  // ParseIntPipe - transforms string to integer
  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number) {
    return `User #${id}`; // id is a number
  }

  // ParseBoolPipe
  @Get()
  findAll(@Query('active', ParseBoolPipe) active: boolean) {
    return active; // boolean
  }

  // ParseArrayPipe
  @Get('by-ids')
  findByIds(
    @Query('ids', new ParseArrayPipe({ items: Number, separator: ',' }))
    ids: number[],
  ) {
    return ids; // [1, 2, 3]
  }

  // ParseUUIDPipe
  @Get('uuid/:id')
  findByUUID(@Param('id', ParseUUIDPipe) id: string) {
    return id; // Valid UUID
  }

  // DefaultValuePipe
  @Get('paginated')
  paginate(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('limit', new DefaultValuePipe(10), ParseIntPipe) limit: number,
  ) {
    return { page, limit };
  }

  // ValidationPipe (with DTO)
  @Post()
  create(@Body(ValidationPipe) createUserDto: CreateUserDto) {
    return createUserDto;
  }
}
```

#### Custom Validation Pipe

```typescript
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParsePositiveIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);

    if (isNaN(val)) {
      throw new BadRequestException('Validation failed: not a number');
    }

    if (val <= 0) {
      throw new BadRequestException('Validation failed: must be positive');
    }

    return val;
  }
}

// Usage
@Get(':id')
findOne(@Param('id', ParsePositiveIntPipe) id: number) {
  return `User #${id}`;
}
```

#### Custom Transformation Pipe

```typescript
@Injectable()
export class TrimPipe implements PipeTransform {
  transform(value: any) {
    if (typeof value === 'string') {
      return value.trim();
    }

    if (typeof value === 'object' && value !== null) {
      return this.transformObject(value);
    }

    return value;
  }

  private transformObject(obj: any): any {
    const result = {};
    for (const [key, value] of Object.entries(obj)) {
      result[key] = typeof value === 'string' ? value.trim() : value;
    }
    return result;
  }
}

// Usage
@Post()
create(@Body(TrimPipe) createUserDto: CreateUserDto) {
  // All string fields are trimmed
  return createUserDto;
}
```

#### Global ValidationPipe Configuration

```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true, // Strip properties not in DTO
      forbidNonWhitelisted: true, // Throw error if extra properties
      transform: true, // Auto-transform payloads to DTO instances
      transformOptions: {
        enableImplicitConversion: true, // Auto-convert types
      },
    }),
  );

  await app.listen(3000);
}
```

---

## Exception Filters

### Simple Explanation

Exception filters catch errors and format them before sending to the client. They're like error handlers that turn ugly errors into nice, consistent error messages.

**Example:** Instead of showing a cryptic database error, show "User with this email already exists".

### Technical Explanation

Exception filters handle all exceptions thrown across the application and allow customizing the error response.

#### Basic Exception Filter

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus
} from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: typeof exceptionResponse === 'string'
        ? exceptionResponse
        : (exceptionResponse as any).message
    });
  }
}

// Apply to route
@Post()
@UseFilters(HttpExceptionFilter)
create(@Body() createUserDto: CreateUserDto) {
  throw new BadRequestException('Invalid data');
}

// Apply globally
app.useGlobalFilters(new HttpExceptionFilter());
```

#### All Exceptions Filter

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    const message =
      exception instanceof HttpException
        ? exception.message
        : 'Internal server error';

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message,
    });
  }
}
```

#### Validation Exception Filter

```typescript
import { BadRequestException } from '@nestjs/common';

@Catch(BadRequestException)
export class ValidationExceptionFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const exceptionResponse: any = exception.getResponse();

    // Format validation errors nicely
    const errors = Array.isArray(exceptionResponse.message)
      ? exceptionResponse.message
      : [exceptionResponse.message];

    response.status(400).json({
      statusCode: 400,
      message: 'Validation failed',
      errors,
    });
  }
}
```

#### Custom Exceptions

```typescript
// Define custom exception
export class UserNotFoundException extends HttpException {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`, HttpStatus.NOT_FOUND);
  }
}

export class DuplicateEmailException extends HttpException {
  constructor(email: string) {
    super(`User with email ${email} already exists`, HttpStatus.CONFLICT);
  }
}

// Service usage
@Injectable()
export class UsersService {
  async findOne(id: string): Promise<User> {
    const user = await this.usersRepository.findOne({ where: { id } });
    if (!user) {
      throw new UserNotFoundException(id);
    }
    return user;
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const existing = await this.findByEmail(createUserDto.email);
    if (existing) {
      throw new DuplicateEmailException(createUserDto.email);
    }
    // Create user...
  }
}
```

---

## Configuration

### Simple Explanation

Configuration manages environment-specific settings (development, staging, production) without hardcoding values. It's like having different settings files for different environments.

**Example:** Database connection string is different in development (localhost) vs production (cloud server).

### Technical Explanation

#### Using @nestjs/config

```bash
npm install @nestjs/config
```

```typescript
// .env file
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=postgres
DATABASE_PASSWORD=password
DATABASE_NAME=mydb
JWT_SECRET=my-secret-key
JWT_EXPIRES_IN=7d

// app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,        // Make ConfigModule global
      envFilePath: '.env',   // Path to .env file
      ignoreEnvFile: false,  // Set to true in production
    })
  ]
})
export class AppModule {}

// Usage in service
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class AppService {
  constructor(private configService: ConfigService) {}

  getDatabaseConfig() {
    return {
      host: this.configService.get<string>('DATABASE_HOST'),
      port: this.configService.get<number>('DATABASE_PORT'),
      user: this.configService.get<string>('DATABASE_USER'),
      password: this.configService.get<string>('DATABASE_PASSWORD'),
      database: this.configService.get<string>('DATABASE_NAME')
    };
  }
}
```

#### Custom Configuration Files

```typescript
// config/database.config.ts
import { registerAs } from '@nestjs/config';

export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: parseInt(process.env.DATABASE_PORT, 10) || 5432,
  username: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD,
  database: process.env.DATABASE_NAME,
  synchronize: process.env.NODE_ENV !== 'production',
}));

// config/jwt.config.ts
export default registerAs('jwt', () => ({
  secret: process.env.JWT_SECRET,
  expiresIn: process.env.JWT_EXPIRES_IN || '7d',
}));

// app.module.ts
import databaseConfig from './config/database.config';
import jwtConfig from './config/jwt.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig, jwtConfig],
      isGlobal: true,
    }),
  ],
})
export class AppModule {}

// Usage
@Injectable()
export class UsersService {
  constructor(private configService: ConfigService) {}

  getConfig() {
    // Access nested config
    const dbHost = this.configService.get<string>('database.host');
    const jwtSecret = this.configService.get<string>('jwt.secret');

    // Or get entire object
    const dbConfig = this.configService.get('database');
  }
}
```

#### Validation Schema

```typescript
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test')
          .default('development'),
        PORT: Joi.number().default(3000),
        DATABASE_HOST: Joi.string().required(),
        DATABASE_PORT: Joi.number().default(5432),
        DATABASE_USER: Joi.string().required(),
        DATABASE_PASSWORD: Joi.string().required(),
        DATABASE_NAME: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
        JWT_EXPIRES_IN: Joi.string().default('7d'),
      }),
    }),
  ],
})
export class AppModule {}
```

---

## Database Integration (TypeORM)

### Simple Explanation

TypeORM is a tool that lets you work with databases using TypeScript classes instead of writing SQL. Your classes become database tables automatically.

**Example:** A `User` class with properties becomes a `users` table with columns.

### Technical Explanation

#### Setup

```bash
npm install @nestjs/typeorm typeorm pg
```

```typescript
// app.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'password',
      database: 'mydb',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // Don't use in production!
      logging: true,
    }),
  ],
})
export class AppModule {}
```

#### Entity

```typescript
// user.entity.ts
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn,
  BeforeInsert,
  BeforeUpdate,
} from 'typeorm';
import * as bcrypt from 'bcrypt';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @Column({ type: 'enum', enum: ['user', 'admin'], default: 'user' })
  role: string;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @BeforeInsert()
  @BeforeUpdate()
  async hashPassword() {
    if (this.password) {
      this.password = await bcrypt.hash(this.password, 10);
    }
  }
}
```

#### Relations

```typescript
// user.entity.ts
import { Entity, Column, PrimaryGeneratedColumn, OneToMany } from 'typeorm';
import { Post } from '../posts/post.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];
}

// post.entity.ts
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  ManyToOne,
  ManyToMany,
  JoinTable,
} from 'typeorm';
import { User } from '../users/user.entity';
import { Tag } from '../tags/tag.entity';

@Entity()
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  title: string;

  @Column('text')
  content: string;

  @ManyToOne(() => User, (user) => user.posts)
  author: User;

  @ManyToMany(() => Tag, (tag) => tag.posts)
  @JoinTable()
  tags: Tag[];
}

// tag.entity.ts
@Entity()
export class Tag {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @ManyToMany(() => Post, (post) => post.tags)
  posts: Post[];
}
```

#### Repository Pattern

```typescript
// users.module.ts
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}

// users.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  // Create
  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.usersRepository.create(createUserDto);
    return this.usersRepository.save(user);
  }

  // Read all
  async findAll(
    page: number = 1,
    limit: number = 10,
  ): Promise<[User[], number]> {
    return this.usersRepository.findAndCount({
      skip: (page - 1) * limit,
      take: limit,
      order: { createdAt: 'DESC' },
    });
  }

  // Read one
  async findOne(id: string): Promise<User> {
    return this.usersRepository.findOne({
      where: { id },
      relations: ['posts'], // Load relations
    });
  }

  // Read by email
  async findByEmail(email: string): Promise<User | null> {
    return this.usersRepository.findOne({ where: { email } });
  }

  // Update
  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    await this.usersRepository.update(id, updateUserDto);
    return this.findOne(id);
  }

  // Delete
  async remove(id: string): Promise<void> {
    await this.usersRepository.delete(id);
  }

  // Complex queries
  async findActiveUsersWithPosts(): Promise<User[]> {
    return this.usersRepository
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.posts', 'post')
      .where('user.isActive = :isActive', { isActive: true })
      .andWhere('post.published = :published', { published: true })
      .getMany();
  }

  // Raw query
  async getUserStats(): Promise<any> {
    return this.usersRepository.query(`
      SELECT 
        COUNT(*) as total_users,
        SUM(CASE WHEN is_active THEN 1 ELSE 0 END) as active_users
      FROM users
    `);
  }
}
```

#### Transactions

```typescript
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, DataSource } from 'typeorm';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
    private dataSource: DataSource,
  ) {}

  async createUserWithProfile(createUserDto: CreateUserDto): Promise<User> {
    const queryRunner = this.dataSource.createQueryRunner();

    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      // Create user
      const user = queryRunner.manager.create(User, createUserDto);
      await queryRunner.manager.save(user);

      // Create profile
      const profile = queryRunner.manager.create(Profile, {
        userId: user.id,
        bio: 'New user',
      });
      await queryRunner.manager.save(profile);

      await queryRunner.commitTransaction();
      return user;
    } catch (err) {
      await queryRunner.rollbackTransaction();
      throw err;
    } finally {
      await queryRunner.release();
    }
  }
}
```

---

## Authentication & Authorization

### Simple Explanation

Authentication verifies who you are (login). Authorization determines what you're allowed to do (permissions).

**Example:**

- Authentication: Logging in with email/password
- Authorization: Only admins can delete users

### Technical Explanation

#### JWT Authentication Setup

```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcrypt
npm install -D @types/passport-jwt @types/bcrypt
```

#### Auth Module

```typescript
// auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { JwtStrategy } from './strategies/jwt.strategy';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    PassportModule,
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { expiresIn: '7d' },
    }),
    UsersModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

#### Auth Service

```typescript
// auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async register(registerDto: RegisterDto) {
    const existingUser = await this.usersService.findByEmail(registerDto.email);
    if (existingUser) {
      throw new ConflictException('Email already exists');
    }

    const user = await this.usersService.create(registerDto);

    return {
      access_token: this.generateToken(user),
      user: this.sanitizeUser(user),
    };
  }

  async login(loginDto: LoginDto) {
    const user = await this.validateUser(loginDto.email, loginDto.password);

    return {
      access_token: this.generateToken(user),
      user: this.sanitizeUser(user),
    };
  }

  async validateUser(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(password, user.password);

    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return user;
  }

  generateToken(user: User): string {
    const payload = {
      sub: user.id,
      email: user.email,
      role: user.role,
    };

    return this.jwtService.sign(payload);
  }

  private sanitizeUser(user: User) {
    const { password, ...result } = user;
    return result;
  }

  async verifyToken(token: string) {
    try {
      return this.jwtService.verify(token);
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }
}
```

#### JWT Strategy

```typescript
// auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { UsersService } from '../../users/users.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(private usersService: UsersService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: any) {
    const user = await this.usersService.findOne(payload.sub);

    if (!user) {
      throw new UnauthorizedException();
    }

    return user; // Attached to request.user
  }
}
```

#### Auth Controller

```typescript
// auth/auth.controller.ts
import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { AuthService } from './auth.service';
import { JwtAuthGuard } from './guards/jwt-auth.guard';

@Controller('auth')
export class AuthController {
  constructor(private authService: AuthService) {}

  @Post('register')
  register(@Body() registerDto: RegisterDto) {
    return this.authService.register(registerDto);
  }

  @Post('login')
  login(@Body() loginDto: LoginDto) {
    return this.authService.login(loginDto);
  }

  @UseGuards(JwtAuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }

  @UseGuards(JwtAuthGuard)
  @Post('refresh')
  refresh(@Request() req) {
    return {
      access_token: this.authService.generateToken(req.user),
    };
  }
}
```

#### Guards

```typescript
// auth/guards/jwt-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

// auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.get<string[]>(
      'roles',
      context.getHandler(),
    );

    if (!requiredRoles) {
      return true;
    }

    const request = context.switchToHttp().getRequest();
    const user = request.user;

    return requiredRoles.some((role) => user.role === role);
  }
}
```

#### Protected Routes

```typescript
@Controller('users')
export class UsersController {
  // Public route
  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  // Protected route - requires authentication
  @UseGuards(JwtAuthGuard)
  @Get('me')
  getProfile(@Request() req) {
    return req.user;
  }

  // Protected route - requires specific role
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin')
  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }

  // Multiple roles
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin', 'moderator')
  @Get('dashboard')
  getDashboard() {
    return 'Dashboard data';
  }
}
```

---

## Testing

### Simple Explanation

Testing ensures your code works as expected. Unit tests check individual functions, integration tests check how components work together, e2e tests check the entire application flow.

### Technical Explanation

#### Unit Testing

```typescript
// users.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { getRepositoryToken } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { Repository } from 'typeorm';

describe('UsersService', () => {
  let service: UsersService;
  let repository: Repository<User>;

  const mockRepository = {
    find: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    save: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: getRepositoryToken(User),
          useValue: mockRepository,
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    repository = module.get<Repository<User>>(getRepositoryToken(User));
  });

  afterEach(() => {
    jest.clearAllMocks();
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('findAll', () => {
    it('should return an array of users', async () => {
      const users = [
        { id: '1', email: 'user1@example.com', name: 'User 1' },
        { id: '2', email: 'user2@example.com', name: 'User 2' },
      ];

      mockRepository.find.mockResolvedValue(users);

      const result = await service.findAll();

      expect(result).toEqual(users);
      expect(mockRepository.find).toHaveBeenCalled();
    });
  });

  describe('findOne', () => {
    it('should return a user if found', async () => {
      const user = { id: '1', email: 'user@example.com', name: 'User' };

      mockRepository.findOne.mockResolvedValue(user);

      const result = await service.findOne('1');

      expect(result).toEqual(user);
      expect(mockRepository.findOne).toHaveBeenCalledWith({
        where: { id: '1' },
      });
    });

    it('should throw NotFoundException if user not found', async () => {
      mockRepository.findOne.mockResolvedValue(null);

      await expect(service.findOne('999')).rejects.toThrow(NotFoundException);
    });
  });

  describe('create', () => {
    it('should create and return a user', async () => {
      const createUserDto = {
        email: 'new@example.com',
        password: 'password123',
      };
      const user = { id: '1', ...createUserDto };

      mockRepository.create.mockReturnValue(user);
      mockRepository.save.mockResolvedValue(user);

      const result = await service.create(createUserDto);

      expect(result).toEqual(user);
      expect(mockRepository.create).toHaveBeenCalledWith(createUserDto);
      expect(mockRepository.save).toHaveBeenCalledWith(user);
    });
  });
});
```

#### Controller Testing

```typescript
// users.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

describe('UsersController', () => {
  let controller: UsersController;
  let service: UsersService;

  const mockUsersService = {
    findAll: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UsersController],
      providers: [
        {
          provide: UsersService,
          useValue: mockUsersService,
        },
      ],
    }).compile();

    controller = module.get<UsersController>(UsersController);
    service = module.get<UsersService>(UsersService);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });

  describe('findAll', () => {
    it('should return an array of users', async () => {
      const users = [{ id: '1', email: 'user@example.com' }];
      mockUsersService.findAll.mockResolvedValue(users);

      const result = await controller.findAll();

      expect(result).toEqual(users);
      expect(service.findAll).toHaveBeenCalled();
    });
  });

  describe('create', () => {
    it('should create a user', async () => {
      const createUserDto = { email: 'new@example.com', password: 'pass123' };
      const user = { id: '1', ...createUserDto };

      mockUsersService.create.mockResolvedValue(user);

      const result = await controller.create(createUserDto);

      expect(result).toEqual(user);
      expect(service.create).toHaveBeenCalledWith(createUserDto);
    });
  });
});
```

#### E2E Testing

```typescript
// test/users.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from './../src/app.module';

describe('UsersController (e2e)', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();

    // Login to get auth token
    const loginResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'admin@example.com', password: 'password123' });

    authToken = loginResponse.body.access_token;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/users (GET)', () => {
    it('should return array of users', () => {
      return request(app.getHttpServer())
        .get('/users')
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });

    it('should support pagination', () => {
      return request(app.getHttpServer())
        .get('/users?page=1&limit=5')
        .expect(200)
        .expect((res) => {
          expect(res.body.length).toBeLessThanOrEqual(5);
        });
    });
  });

  describe('/users (POST)', () => {
    it('should create a new user', () => {
      const createUserDto = {
        email: 'test@example.com',
        password: 'password123',
        firstName: 'Test',
        lastName: 'User',
      };

      return request(app.getHttpServer())
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(createUserDto)
        .expect(201)
        .expect((res) => {
          expect(res.body.email).toBe(createUserDto.email);
          expect(res.body).not.toHaveProperty('password');
        });
    });

    it('should return 400 for invalid email', () => {
      const createUserDto = {
        email: 'invalid-email',
        password: 'password123',
      };

      return request(app.getHttpServer())
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .send(createUserDto)
        .expect(400);
    });

    it('should return 401 without auth token', () => {
      return request(app.getHttpServer())
        .post('/users')
        .send({ email: 'test@example.com', password: 'pass123' })
        .expect(401);
    });
  });

  describe('/users/:id (GET)', () => {
    it('should return a specific user', () => {
      return request(app.getHttpServer())
        .get('/users/1')
        .expect(200)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body).toHaveProperty('email');
        });
    });

    it('should return 404 for non-existent user', () => {
      return request(app.getHttpServer()).get('/users/999999').expect(404);
    });
  });

  describe('/users/:id (DELETE)', () => {
    it('should delete a user', () => {
      return request(app.getHttpServer())
        .delete('/users/1')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
    });

    it('should return 401 without auth token', () => {
      return request(app.getHttpServer()).delete('/users/1').expect(401);
    });
  });
});
```

---

## Best Practices

### 1. Module Organization

```
src/
├── app.module.ts
├── main.ts
├── common/                 # Shared utilities
│   ├── decorators/
│   ├── guards/
│   ├── filters/
│   ├── interceptors/
│   └── pipes/
├── config/                 # Configuration files
│   ├── database.config.ts
│   └── jwt.config.ts
├── users/                  # Feature module
│   ├── dto/
│   │   ├── create-user.dto.ts
│   │   └── update-user.dto.ts
│   ├── entities/
│   │   └── user.entity.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.module.ts
│   └── users.service.spec.ts
└── auth/
    ├── strategies/
    ├── guards/
    ├── auth.controller.ts
    ├── auth.service.ts
    └── auth.module.ts
```

### 2. Error Handling

```typescript
// Use specific exceptions
throw new NotFoundException(`User with ID ${id} not found`);
throw new BadRequestException('Invalid input data');
throw new UnauthorizedException('Invalid credentials');
throw new ForbiddenException('Insufficient permissions');
throw new ConflictException('Email already exists');

// Create custom exceptions
export class UserNotFoundException extends NotFoundException {
  constructor(userId: string) {
    super(`User with ID ${userId} not found`);
  }
}
```

### 3. DTOs and Validation

```typescript
// Always use DTOs for input validation
// Always use ValidationPipe globally
// Use class-validator decorators
// Use PartialType for update DTOs
```

### 4. Dependency Injection

```typescript
// Use constructor injection
// Don't use property injection
// Make dependencies explicit
// Use interfaces for better testability
```

### 5. Async/Await

```typescript
// Always use async/await for async operations
// Handle errors properly with try-catch
// Return Promises from service methods
```

---

## Common Interview Questions

### 1. What is the difference between @Injectable() and @Controller()?

**Simple:** @Injectable() marks a class that can be injected as a dependency (services). @Controller() marks a class that handles HTTP requests.

**Technical:** @Injectable() registers a provider in the DI container. @Controller() defines a request handler with route mapping and HTTP method decorators.

### 2. What is dependency injection and why use it?

**Simple:** Instead of creating dependencies yourself, NestJS provides them automatically. This makes code testable and maintainable.

**Technical:** DI is an IoC pattern where dependencies are injected at runtime rather than compiled time. Benefits: loose coupling, testability, maintainability, reusability.

### 3. What's the difference between Guard and Middleware?

**Simple:**

- Middleware runs before route handler, can modify request/response
- Guard determines if request should be handled (true/false)

**Technical:**

- Middleware: Executed before route handler, has access to req/res, can call next()
- Guard: Implements CanActivate, returns boolean, executed after middleware but before interceptors

### 4. Explain the request lifecycle in NestJS

**Order:**

1. Middleware
2. Guards
3. Interceptors (before)
4. Pipes
5. Controller/Route handler
6. Interceptors (after)
7. Exception filters

### 5. When would you use useClass vs useValue vs useFactory?

**useClass:** When you want to provide a different implementation of a class
**useValue:** When you want to provide a static value/object
**useFactory:** When you need dynamic creation with dependencies

---
