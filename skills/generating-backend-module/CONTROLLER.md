# Template de Controller e Module (NestJS)

Parte da skill [generating-backend-module](SKILL.md).

```typescript
import {
  Body, Controller, Get, Param, ParseIntPipe, Patch, Post, Put, Query, UseGuards,
} from '@nestjs/common';
import {
  ApiBearerAuth, ApiBody, ApiOperation, ApiParam, ApiResponse, ApiTags,
} from '@nestjs/swagger';
import { ApiResponse as ApiEnvelope } from '@org/shared';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RequirePermissions } from '../auth/decorators/require-permissions.decorator';
import { Create{Entity}Dto } from './dto/create-{module}.dto';
import { Update{Entity}Dto } from './dto/update-{module}.dto';
import { {Entity}ResponseDto } from './dto/{module}-response.dto';
import { {Entity}Service } from './{module}.service';

@ApiTags('{module}')
@ApiBearerAuth('JWT-auth')
@UseGuards(JwtAuthGuard)
@Controller('{module}')
export class {Entity}Controller {
  constructor(private readonly service: {Entity}Service) {}

  @Get()
  @RequirePermissions('{module}:read')
  @ApiOperation({ summary: 'Listar {entityLabel}', description: 'Retorna todos os registros.' })
  @ApiResponse({ status: 200, type: [{Entity}ResponseDto] })
  findAll(@Query('tenantId') tenantId?: number): Promise<ApiEnvelope<{Entity}ResponseDto[]>> {
    return this.service.findAll(tenantId);
  }

  @Get(':id')
  @RequirePermissions('{module}:read')
  @ApiOperation({ summary: 'Buscar {entityLabel} por ID' })
  @ApiParam({ name: 'id', type: Number })
  @ApiResponse({ status: 200, type: {Entity}ResponseDto })
  @ApiResponse({ status: 404, description: 'Nao encontrado' })
  findOne(@Param('id', ParseIntPipe) id: number): Promise<ApiEnvelope<{Entity}ResponseDto>> {
    return this.service.findOne(id);
  }

  @Post()
  @RequirePermissions('{module}:create')
  @ApiOperation({ summary: 'Criar {entityLabel}' })
  @ApiBody({ type: Create{Entity}Dto })
  @ApiResponse({ status: 201, type: {Entity}ResponseDto })
  @ApiResponse({ status: 400, description: 'Dados invalidos' })
  create(@Body() dto: Create{Entity}Dto): Promise<ApiEnvelope<{Entity}ResponseDto>> {
    return this.service.create(dto);
  }

  @Put(':id')
  @RequirePermissions('{module}:update')
  @ApiOperation({ summary: 'Atualizar {entityLabel}' })
  @ApiParam({ name: 'id', type: Number })
  @ApiBody({ type: Update{Entity}Dto })
  @ApiResponse({ status: 200, type: {Entity}ResponseDto })
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() dto: Update{Entity}Dto,
  ): Promise<ApiEnvelope<{Entity}ResponseDto>> {
    return this.service.update(id, dto);
  }

  @Patch(':id/status')
  @RequirePermissions('{module}:update')
  @ApiOperation({ summary: 'Alternar status de {entityLabel}' })
  @ApiParam({ name: 'id', type: Number })
  @ApiResponse({ status: 200, type: {Entity}ResponseDto })
  toggleStatus(@Param('id', ParseIntPipe) id: number): Promise<ApiEnvelope<{Entity}ResponseDto>> {
    return this.service.toggleStatus(id);
  }
}
```

## Module

```typescript
import { Module } from '@nestjs/common';
import { {Entity}Controller } from './{module}.controller';
import { {Entity}Service } from './{module}.service';

@Module({
  controllers: [{Entity}Controller],
  providers: [{Entity}Service],
  exports: [{Entity}Service],
})
export class {Entity}Module {}
```

Registrar no modulo raiz em `imports: [{Entity}Module]`.

## Notas de adaptacao

- `JwtAuthGuard` e `RequirePermissions` sao os nomes convencionais aqui; trocar pelos
  equivalentes do projeto destino (passport, Firebase, Auth0, custom).
- Se o projeto nao tiver RBAC, remover `@RequirePermissions` e manter apenas o guard de auth.
- O alias `ApiEnvelope` evita colisao entre o `ApiResponse` do Swagger e o wrapper de
  resposta da aplicacao. Manter esse padrao de rename em todo controller.
