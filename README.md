# Postgrape

Postgrape is a TypeScript framework for robust, transactional PostgreSQL data access, designed for use with NestJS. It provides generic repositories, advanced query options, transaction decorators, and seamless integration with Luxon for date/time handling.

## Features

- **Transactional PostgreSQL Client**: `PostgrapeClient` wraps a PostgreSQL client, supporting transactions, savepoints, and release
- **Connection Pool Management**: `PostgrapeClientFactory` manages connections and client acquisition
- **Generic Repository**: `Repository<TEntity>` provides CRUD operations and flexible query building
- **Searchable Repository**: `SearchableRepository<TEntity>` adds full-text search across columns
- **Batch Insert**: `MultipleRepository<TEntity>` supports batch creation via stored procedures
- **NestJS Integration**: `PostgrapeModule` and decorators for automatic transaction management in services
- **Advanced Query Operators**: Symbol-based SQL operators for expressive queries (`Op`).
- **Luxon Integration**: Handles PostgreSQL date/time types with Luxon `DateTime`

## Installation

Add to your project:

```sh
npm install postgrape pg luxon
```

## Usage

### 1. Define an Entity and Data Client

```typescript
import { Entity, Repository, PostgrapeClient } from 'postgrape';

export interface Warehouse extends Entity {
  id: number;
  name: string;
  address: string;
  lat: number;
  lng: number;
  openingTime: Duration;
  createdAt: DurationWithTZ;
  // ...other fields
}

export class DataClient extends PostgrapeClient {
  public warehouses = new Repository<Warehouse>({ table: 'warehouse' }, this._config);
}
```

### 2. Integrate with NestJS

```typescript
import { PostgrapeModule } from 'postgrape';

@Module({
  imports: [
    PostgrapeModule.register({
      host: 'localhost',
      port: 5432,
      database: 'mydb',
      user: 'postgres',
      password: 'secret',
      dataClient: DataClient,
    }),
  ],
})
export class AppModule {}
```

### 3. Extend TransactableService and utilize Transaction Decorators

```typescript
import { Transaction, DC, TransactableService } from 'postgrape';

class MyService extends TransactableService {
  @Transaction()
  async doSomething(@DC() client: DataClient) {
    // All operations here are wrapped in a transaction
    await client.warehouses.create({
      name: 'ATB Market Warehouse',
      address: 'Some address',
      lat: 48.55,
      lng: 35.09,
      openingTime: Duration.fromObject({ hours: 8 }),
      createdAt: DateTime.now(),
      // ...other fields
    });
  }
}
```

### 4. Querying with Operators

```typescript
import { Op } from 'postgrape';

const results = await client.warehouses.find({
  where: {
    lat: { [Op.gt]: 40 },
    name: ['ATB', 'Silpo'],
    [Op.or]: [{ id: 1 }, { id: 2 }],
  },
  orderBy: { id: 'DESC' },
  limit: 10,
});
```

### 5. Full-Text Search

```typescript
import { SearchableRepository } from 'postgrape';

const searchResults = await client.warehouses.search('market', {
  columnsForSearch: ['name', 'address'],
  limit: 5,
});
```

## API Reference

- `PostgrapeClient`: Transactional client, extend to add repositories
- `PostgrapeClientFactory`: Connection pool, client acquisition
- `Repository<TEntity>`: CRUD, query, flexible options
- `SearchableRepository<TEntity>`: Full-text search
- `MultipleRepository<TEntity>`: Batch insert
- `Transaction`, `DC`, `TransactableService`: Transaction decorators for NestJS services
- `DurationWithTZ`: Luxon-based time with timezone
- `Op`: SQL operators for queries
- `InvalidArgumentsException`: Error handling
