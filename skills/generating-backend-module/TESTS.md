# Template de Spec do Service (TDD)

Parte da skill [generating-backend-module](SKILL.md).

**Escrever ANTES do service.** Ciclo Red > Green > Refactor.

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { getLoggerToken } from 'nestjs-pino';
import { {Entity}Service } from './{module}.service';
import { PrismaService } from '../prisma/prisma.service';

describe('{Entity}Service', () => {
  let service: {Entity}Service;
  let prisma: jest.Mocked<PrismaService>;

  const mockItem = {
    id: 1,
    name: 'Test {Entity}',
    description: 'Description',
    status: true,
    tenant_id: 1,
  };

  const mockLogger = {
    info: jest.fn(),
    error: jest.fn(),
    warn: jest.fn(),
    debug: jest.fn(),
  };

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        {Entity}Service,
        {
          provide: PrismaService,
          useValue: {
            {model}: {
              findMany: jest.fn(),
              findUnique: jest.fn(),
              create: jest.fn(),
              update: jest.fn(),
            },
          },
        },
        {
          provide: getLoggerToken({Entity}Service.name),
          useValue: mockLogger,
        },
      ],
    }).compile();

    service = module.get<{Entity}Service>({Entity}Service);
    prisma = module.get(PrismaService);
  });

  describe('findAll', () => {
    it('deve retornar lista de {entityLabel}', async () => {
      (prisma.{model}.findMany as jest.Mock).mockResolvedValue([mockItem]);

      const result = await service.findAll();

      expect(result.statusCode).toBe(200);
      expect(result.data).toHaveLength(1);
      expect(result.data[0].name).toBe('Test {Entity}');
    });

    it('deve filtrar por tenantId', async () => {
      (prisma.{model}.findMany as jest.Mock).mockResolvedValue([mockItem]);

      await service.findAll(1);

      expect(prisma.{model}.findMany).toHaveBeenCalledWith(
        expect.objectContaining({ where: { tenant_id: 1 } }),
      );
    });
  });

  describe('findOne', () => {
    it('deve retornar {entityLabel} por id', async () => {
      (prisma.{model}.findUnique as jest.Mock).mockResolvedValue(mockItem);

      const result = await service.findOne(1);

      expect(result.statusCode).toBe(200);
      expect(result.data.id).toBe(1);
    });

    it('deve lancar excecao quando nao encontrado', async () => {
      (prisma.{model}.findUnique as jest.Mock).mockResolvedValue(null);

      await expect(service.findOne(999)).rejects.toThrow();
    });
  });

  describe('create', () => {
    it('deve criar {entityLabel}', async () => {
      (prisma.{model}.create as jest.Mock).mockResolvedValue(mockItem);
      const dto = { name: 'Test', description: 'Desc', tenantId: 1 };

      const result = await service.create(dto as Create{Entity}Dto);

      expect(result.statusCode).toBe(201);
      expect(prisma.{model}.create).toHaveBeenCalled();
    });
  });

  describe('toggleStatus', () => {
    it('deve inverter status', async () => {
      (prisma.{model}.findUnique as jest.Mock).mockResolvedValue({ status: true });
      (prisma.{model}.update as jest.Mock).mockResolvedValue({ ...mockItem, status: false });

      const result = await service.toggleStatus(1);

      expect(prisma.{model}.update).toHaveBeenCalledWith(
        expect.objectContaining({ data: { status: false } }),
      );
      expect(result.statusCode).toBe(200);
    });
  });
});
```

## Cobertura minima

Por metodo publico: um caso feliz e um caso de erro. Alem disso:

- Filtro/escopo aplicado corretamente na query (assert no argumento do Prisma, nao so no retorno).
- Mapeamento `snake_case` > `camelCase` no DTO de saida.
- Validacao de id invalido (0, negativo, `NaN`).
- Nao testar internals do Prisma nem do NestJS: mockar a fronteira e verificar o contrato.
