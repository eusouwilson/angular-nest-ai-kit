# Template de Service (NestJS + Prisma)

Parte da skill [generating-backend-module](SKILL.md).

```typescript
import { Injectable } from '@nestjs/common';
import { InjectPinoLogger, PinoLogger } from 'nestjs-pino';
import { ApiResponse } from '@org/shared';
import { PrismaService } from '../prisma/prisma.service';
import { BaseService } from '../common/base/base.service';
import { Create{Entity}Dto } from './dto/create-{module}.dto';
import { Update{Entity}Dto } from './dto/update-{module}.dto';
import { {Entity}ResponseDto } from './dto/{module}-response.dto';

@Injectable()
export class {Entity}Service extends BaseService {
  constructor(
    protected readonly prisma: PrismaService,
    @InjectPinoLogger({Entity}Service.name)
    protected readonly logger: PinoLogger,
  ) {
    super(prisma, logger);
  }

  async findAll(tenantId?: number): Promise<ApiResponse<{Entity}ResponseDto[]>> {
    try {
      const where = tenantId ? { tenant_id: tenantId } : {};
      const items = await this.prisma.{model}.findMany({
        where,
        select: {
          id: true,
          name: true,
          description: true,
          status: true,
          tenant_id: true,
        },
        orderBy: { name: 'asc' },
      });
      const data = items.map((item) => this.mapToDto(item));
      return { message: 'Registros listados com sucesso', statusCode: 200, type: 'success', data };
    } catch (error) {
      this.handleServiceError(error, 'listar {entityLabel}');
    }
  }

  async findOne(id: number): Promise<ApiResponse<{Entity}ResponseDto>> {
    this.validateId(id);
    try {
      const item = await this.prisma.{model}.findUnique({ where: { id } });
      this.validateExists(item, '{EntityLabel}', id);
      return { message: 'Registro encontrado', statusCode: 200, type: 'success', data: this.mapToDto(item) };
    } catch (error) {
      this.handleServiceError(error, 'buscar {entityLabel}');
    }
  }

  async create(dto: Create{Entity}Dto): Promise<ApiResponse<{Entity}ResponseDto>> {
    try {
      const item = await this.prisma.{model}.create({
        data: {
          name: dto.name,
          description: dto.description,
          status: dto.status ?? true,
          tenant_id: dto.tenantId,
        },
      });
      return { message: '{EntityLabel} criado com sucesso', statusCode: 201, type: 'success', data: this.mapToDto(item) };
    } catch (error) {
      this.handleServiceError(error, 'criar {entityLabel}');
    }
  }

  async update(id: number, dto: Update{Entity}Dto): Promise<ApiResponse<{Entity}ResponseDto>> {
    this.validateId(id);
    try {
      await this.findOne(id); // valida existencia
      const item = await this.prisma.{model}.update({
        where: { id },
        data: {
          ...(dto.name && { name: dto.name }),
          ...(dto.description && { description: dto.description }),
          ...(dto.status !== undefined && { status: dto.status }),
        },
      });
      return { message: '{EntityLabel} atualizado com sucesso', statusCode: 200, type: 'success', data: this.mapToDto(item) };
    } catch (error) {
      this.handleServiceError(error, 'atualizar {entityLabel}');
    }
  }

  async toggleStatus(id: number): Promise<ApiResponse<{Entity}ResponseDto>> {
    this.validateId(id);
    try {
      const current = await this.prisma.{model}.findUnique({ where: { id }, select: { status: true } });
      this.validateExists(current, '{EntityLabel}', id);
      const item = await this.prisma.{model}.update({
        where: { id },
        data: { status: !current.status },
      });
      return { message: 'Status alterado com sucesso', statusCode: 200, type: 'success', data: this.mapToDto(item) };
    } catch (error) {
      this.handleServiceError(error, 'alterar status');
    }
  }

  private mapToDto(item: {
    id: number;
    name: string;
    description: string;
    status: boolean;
    tenant_id: number;
  }): {Entity}ResponseDto {
    return {
      id: item.id,
      name: item.name,
      description: item.description,
      status: item.status,
      tenantId: item.tenant_id,
    };
  }
}
```

## Pontos de atencao

- `select` explicito em toda query: evita vazar coluna nova sem revisao e reduz payload.
- Toggle de status le o valor atual antes de inverter: nao confiar em valor vindo do cliente.
- Delete fisico so quando o dominio exige; o default aqui e desativacao logica.
- Erros passam sempre por `handleServiceError`, que traduz erro do Prisma para `HttpException`.
- Se o projeto nao for multi-tenant, remover `tenantId` / `tenant_id` do service inteiro.
