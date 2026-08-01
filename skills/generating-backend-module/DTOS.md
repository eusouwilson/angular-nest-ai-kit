# Templates de DTOs (NestJS)

Parte da skill [generating-backend-module](SKILL.md).

## Create DTO

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { IsBoolean, IsNotEmpty, IsOptional, IsString, MinLength } from 'class-validator';

export class Create{Entity}Dto {
  @ApiProperty({ example: 'Nome do registro', description: 'Nome' })
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  name: string;

  @ApiProperty({ example: 'Descricao detalhada', description: 'Descricao' })
  @IsString()
  @IsNotEmpty()
  @MinLength(3)
  description: string;

  @ApiProperty({ example: true, description: 'Status ativo/inativo' })
  @IsBoolean()
  @IsOptional()
  status?: boolean;

  // Campo de escopo multi-tenant. Remover se o projeto nao for multi-tenant.
  @ApiProperty({ example: 1, description: 'Identificador do tenant' })
  @IsNotEmpty()
  tenantId: number;
}
```

Toda propriedade precisa de `@ApiProperty` com `example`: e o que torna o Swagger utilizavel.

## Update DTO

```typescript
import { PartialType } from '@nestjs/swagger';
import { Create{Entity}Dto } from './create-{module}.dto';

export class Update{Entity}Dto extends PartialType(Create{Entity}Dto) {}
```

`PartialType` de `@nestjs/swagger` (nao de `@nestjs/mapped-types`): preserva os metadados
do Swagger no schema gerado.

## Response DTO

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { Expose } from 'class-transformer';
import { I{Entity} } from '@org/shared';

export class {Entity}ResponseDto implements I{Entity} {
  @ApiProperty({ example: 1 })
  @Expose()
  id: number;

  @ApiProperty({ example: 'Nome do registro' })
  @Expose()
  name: string;

  @ApiProperty({ example: 'Descricao' })
  @Expose()
  description: string;

  @ApiProperty({ example: true })
  @Expose()
  status: boolean;

  @ApiProperty({ example: 1 })
  @Expose()
  tenantId: number;
}
```

O response DTO implementa a interface publica que vive na lib compartilhada. Se a interface
ainda nao existe em `@org/shared`, cria-la primeiro: o DTO local nunca e a fonte do contrato.

`@Expose` so tem efeito com `excludeExtraneousValues: true` na serializacao, entao
configurar o `ClassSerializerInterceptor` ou o `plainToInstance` correspondente.
