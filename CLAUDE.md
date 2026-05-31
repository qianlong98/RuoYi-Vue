# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RuoYi-Vue is a Chinese rapid development platform based on Spring Boot + Vue (前后端分离架构). Version 3.9.2 using Spring Boot 4.0.3 with Java 17.

## Build and Run Commands

### Backend (Java/Spring Boot)

```bash
# Build the project (skip tests)
mvn clean package -Dmaven.test.skip=true

# Run the application
java -jar ruoyi-admin/target/ruoyi-admin.jar

# Or use the provided scripts
./ry.sh start      # Linux/Mac
ry.bat             # Windows (interactive menu)
```

Backend runs on port 8080 by default. Requires MySQL and Redis to be running.

### Frontend (Vue)

```bash
cd ruoyi-ui

# Install dependencies
npm install --registry=https://registry.npmmirror.com

# Development server
npm run dev

# Build for production
npm run build:prod

# Build for staging
npm run build:stage
```

Frontend runs on port 80 by default in development mode.

## Module Architecture

Multi-module Maven project with clear separation of concerns:

| Module | Purpose |
|--------|---------|
| **ruoyi-admin** | Application entry point (`RuoYiApplication.java`), REST controllers for system/monitor endpoints |
| **ruoyi-common** | Shared utilities: annotations (`@Log`, `@DataScope`, `@Excel`), constants, domain entities (`AjaxResult`, `BaseEntity`), exceptions, utils (StringUtils, DateUtils, SecurityUtils) |
| **ruoyi-framework** | Configuration layer: SecurityConfig, RedisConfig, DruidConfig, MyBatisConfig; AOP aspects for logging, data scope, rate limiting; JWT authentication filter |
| **ruoyi-system** | Core business: user/role/menu/dept/post/dict/config services and mappers |
| **ruoyi-generator** | Code generator using Velocity templates - generates CRUD code from database tables |
| **ruoyi-quartz** | Scheduled task management with Quartz scheduler |

## Key Architectural Patterns

### Authentication Flow
- JWT-based authentication via `JwtAuthenticationTokenFilter` in framework module
- `TokenService` handles token generation/validation, stored in Redis
- `LoginUser` object cached in Redis with key pattern `login_tokens:{userId}`

### Permission Control
- `@DataScope` annotation for row-level data permission filtering (data_scope aspect intercepts mapper queries)
- `@Log` annotation for operation logging (logs to `sys_oper_log` table)
- Menu/button permissions checked via `PermissionService.hasPermi()`

### Response Format
- All API responses use `AjaxResult` class with standard structure: `{code, msg, data}`
- Success: `AjaxResult.success()` / Error: `AjaxResult.error()`
- Page queries return `TableDataInfo` with `{total, rows, code}`

### Controller Pattern
- Controllers extend `BaseController` which provides pagination helpers (`startPage()`, `getDataTable()`)
- Standard CRUD pattern: `@GetMapping("/list")`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`

## Configuration Files

- `ruoyi-admin/src/main/resources/application.yml` - Main config (server, redis, token, mybatis, pagehelper)
- `ruoyi-admin/src/main/resources/application-druid.yml` - Database connection (MySQL datasource)
- `sql/ry_20260417.sql` - Main database schema and initial data
- `sql/quartz.sql` - Quartz scheduler tables

## Frontend Structure

```
ruoyi-ui/src/
├── api/          # API request modules (system.js, login.js, etc.)
├── components/   # Reusable components (Pagination, FileUpload, DictTag)
├── layout/       # Layout components (Navbar, Sidebar, TagsView)
├── router/       # Dynamic route configuration based on user permissions
├── store/        # Vuex modules (user, app, permission)
├── utils/        # Utilities (request.js with axios interceptors, auth.js for token)
├── views/        # Page components organized by module (system/, monitor/, tool/)
```

## Database Notes

- Default MySQL database name: `ry-vue` (configured in application-druid.yml)
- MyBatis mapper XMLs located at `classpath*:mapper/**/*Mapper.xml`
- Entity classes use `BaseEntity` as parent with common fields (createBy, createTime, updateBy, updateTime)

## Code Generation

Access via `/tool/gen` endpoint. Generates:
- Java entity, mapper, service, controller
- Vue page components
- SQL menu insert statements
Uses Velocity templates in `ruoyi-generator/src/main/resources/vm/`