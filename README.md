# Nexa Store — Diccionario de datos

Base de datos: `nexa_store_db` · Motor: PostgreSQL · Esquema: `public` · Fecha: 2026-08-24

---

## 1. Resumen de tablas

| Tabla | Descripcion | Registros | PK | Tipo PK | FKs |
|-------|-------------|-----------|----|---------|-----|
| `pais` | Paises | 20 | `id_pais` | `double precision` | — |
| `producto_familia` | Familias de producto | 4 | `id_producto_familia` | `double precision` | — |
| `tienda` | Tiendas / sucursales | 10 | `id_tienda` | `double precision` | — |
| `ciudad` | Ciudades | 75 | `id_ciudad` | `integer` | 1 → pais |
| `producto_departamento` | Departamentos de producto | 4 | `id_producto_departamento` | `integer` | 1 → producto_familia |
| `vendedor` | Vendedores | 25 | `id_vendedor` | `integer` | 1 → tienda |
| `cliente` | Clientes | 92 | `id_cliente` | `integer` | 1 → ciudad |
| `producto_categoria` | Categorias de producto | 7 | `id_producto_categoria` | `double precision` | 1 → producto_departamento |
| `producto` | Productos | 109 | `id_producto` | `integer` | 1 → producto_categoria |
| `venta` | Ventas | 252 750 | `id_venta` | `integer` | 3 → tienda, cliente, vendedor |
| `venta_detalle` | Detalle de ventas | 311 963 | `id_venta_detalle` | `integer` | 2 → venta, producto |

---

## 2. Diagrama de relaciones

```mermaid
pais ──1:N──> ciudad ──1:N──> cliente ──1:N──> venta ──1:N──> venta_detalle
                                                          N:1──> producto ──N:1──> producto_categoria
tienda ──1:N──> vendedor ──1:N──> venta                         N:1──> producto_departamento
               tienda ──1:N──> venta                                N:1──> producto_familia
```

### Jerarquia de producto

```
producto_familia
  └── producto_departamento
        └── producto_categoria
              └── producto
```

---

## 3. Diccionario de columnas por tabla

### 3.1 pais

| Columna | Tipo SQL | Nulo | PK | Descripcion |
|---------|----------|------|----|-------------|
| `id_pais` | `double precision` | NO | Si | Identificador unico |
| `descripcion` | `text` | Si | — | Nombre del pais |
| `territorio` | `text` | Si | — | Territorio o region |

Constraint PK: `pais_pkey` · Indices: `idx_pais_lookup` (id_pais, descripcion, territorio)

---

### 3.2 producto_familia

| Columna | Tipo SQL | Nulo | PK | Descripcion |
|---------|----------|------|----|-------------|
| `id_producto_familia` | `double precision` | NO | Si | Identificador unico |
| `descripcion` | `text` | Si | — | Nombre de la familia |

Constraint PK: `producto_familia_pkey` · Indices: `idx_producto_familia_lookup` (id, descripcion)

---

### 3.3 tienda

| Columna | Tipo SQL | Nulo | PK | Descripcion |
|---------|----------|------|----|-------------|
| `id_tienda` | `double precision` | NO | Si | Identificador unico |
| `descripcion` | `text` | Si | — | Nombre de la tienda |

Constraint PK: `tienda_pkey` · Indices: `idx_tienda_lookup` (id, descripcion)

---

### 3.4 ciudad

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_ciudad` | `integer` | NO | Si | — | Identificador unico |
| `nombre` | `text` | Si | — | — | Nombre de la ciudad |
| `departamento` | `text` | Si | — | — | Departamento / provincia |
| `id_pais` | `integer` | Si | — | `pais.id_pais` | Pais al que pertenece |

Constraint PK: `ciudad_pkey` · FK: `pais_fk` → pais(id_pais)
Indices: `idx_ciudad_lookup` (id, nombre, departamento, id_pais)

> Nota: `id_pais` es `integer` en DB, pero la PK de pais es `double precision`.

---

### 3.5 producto_departamento

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_producto_departamento` | `integer` | NO | Si | — | Identificador unico |
| `id_producto_familia` | `integer` | Si | — | `producto_familia.id_producto_familia` | Familia padre |
| `descripcion` | `text` | Si | — | — | Nombre del departamento |

Constraint PK: `producto_departamento_pkey` · FK: `familia_fk` → producto_familia(id)
Indices: `idx_producto_departamento_lookup` (id, id_familia, descripcion)

> Nota: `id_producto_familia` es `integer` en DB, pero la PK de producto_familia es `double precision`.

---

### 3.6 vendedor

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_vendedor` | `integer` | NO | Si | — | Identificador unico |
| `nombre` | `varchar(80)` | NO | — | — | Nombre del vendedor |
| `apellido` | `varchar(80)` | NO | — | — | Apellido del vendedor |
| `edad` | `integer` | NO | — | — | Edad |
| `id_tienda` | `integer` | NO | — | `tienda.id_tienda` | Tienda asignada |
| `sexo` | `varchar` | Si | — | — | Sexo |

Constraint PK: `vendedor_pkey` · FK: `tienda_fk` → tienda(id) **NOT NULL**

> Nota: `id_tienda` es `integer` en DB, pero la PK de tienda es `double precision`.

---

### 3.7 cliente

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_cliente` | `integer` | NO | Si | — | Identificador unico |
| `id_ciudad` | `integer` | Si | — | `ciudad.id_ciudad` | Ciudad de residencia |
| `nombre` | `text` | Si | — | — | Nombre del cliente |
| `direccion` | `text` | Si | — | — | Direccion |

Constraint PK: `cliente_pkey` · FK: `ciudad_fk` → ciudad(id)
Indices: `idx_cliente_lookup` (id, id_ciudad, nombre, direccion)

---

### 3.8 producto_categoria

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_producto_categoria` | `double precision` | NO | Si | — | Identificador unico |
| `descripcion` | `text` | Si | — | — | Nombre de la categoria |
| `id_producto_departamento` | `integer` | Si | — | `producto_departamento.id_producto_departamento` | Departamento padre |

Constraint PK: `producto_categoria_pkey` · FK: `departamento_fk` → producto_departamento(id)

---

### 3.9 producto

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_producto` | `integer` | NO | Si | — | Identificador unico |
| `descripcion` | `text` | Si | — | — | Nombre del producto |
| `id_producto_categoria` | `integer` | Si | — | `producto_categoria.id_producto_categoria` | Categoria padre |
| `precio_costo` | `numeric(15,2)` | Si | — | — | Precio de costo |

Constraint PK: `producto_pkey` · FK: `producto_cat_fk` → producto_categoria(id)
Indices: `idx_producto_lookup` (id, descripcion, id_categoria)

> Nota: `id_producto_categoria` es `integer` en DB, pero la PK de producto_categoria es `double precision`.

---

### 3.10 venta

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_venta` | `integer` | NO | Si | — | Identificador unico |
| `id_tienda` | `integer` | NO | — | `tienda.id_tienda` | Tienda de la venta |
| `id_cliente` | `integer` | NO | — | `cliente.id_cliente` | Cliente comprador |
| `id_vendedor` | `integer` | NO | — | `vendedor.id_vendedor` | Vendedor |
| `fecha_venta` | `date` | NO | — | — | Fecha de la venta |
| `estado` | `varchar` | Si | — | — | Estado (pendiente, completada) |

Constraint PK: `venta_pkey`
FKs: `tienda_fk` → tienda(id), `cliente_fk` → cliente(id), `vendedor_fk` → vendedor(id) — **todas NOT NULL**
Indices: `idx_venta_lookup` (id_venta)

> Nota: Las 3 FKs son `integer` en DB, pero las PKs de tienda y vendedor son `double precision`.

---

### 3.11 venta_detalle

| Columna | Tipo SQL | Nulo | PK | FK | Descripcion |
|---------|----------|------|----|----|-------------|
| `id_venta_detalle` | `integer` | NO | Si | — | Identificador unico |
| `id_venta` | `integer` | Si | — | `venta.id_venta` | Venta padre |
| `id_producto` | `integer` | Si | — | `producto.id_producto` | Producto vendido |
| `unidades_vendidas` | `integer` | Si | — | — | Cantidad de unidades |
| `valor_vendido` | `numeric(18,2)` | Si | — | — | Valor total vendido |

Constraint PK: `venta_detalle_pkey`
FKs: `venta_fk` → venta(id), `producto_fk` → producto(id)
Indices: `idx_venta_detalle_lookup` (id_venta_detalle)

---

## 4. Restricciones (constraints)

| Constraint | Tipo | Tabla | Columna | Referencia |
|------------|------|-------|---------|------------|
| `pais_pkey` | PRIMARY KEY | pais | id_pais | — |
| `producto_familia_pkey` | PRIMARY KEY | producto_familia | id_producto_familia | — |
| `tienda_pkey` | PRIMARY KEY | tienda | id_tienda | — |
| `ciudad_pkey` | PRIMARY KEY | ciudad | id_ciudad | — |
| `pais_fk` | FOREIGN KEY | ciudad | id_pais | → pais(id_pais) |
| `producto_departamento_pkey` | PRIMARY KEY | producto_departamento | id_producto_departamento | — |
| `familia_fk` | FOREIGN KEY | producto_departamento | id_producto_familia | → producto_familia(id) |
| `vendedor_pkey` | PRIMARY KEY | vendedor | id_vendedor | — |
| `tienda_fk` | FOREIGN KEY | vendedor | id_tienda | → tienda(id) |
| `cliente_pkey` | PRIMARY KEY | cliente | id_cliente | — |
| `ciudad_fk` | FOREIGN KEY | cliente | id_ciudad | → ciudad(id) |
| `producto_categoria_pkey` | PRIMARY KEY | producto_categoria | id_producto_categoria | — |
| `departamento_fk` | FOREIGN KEY | producto_categoria | id_producto_departamento | → producto_departamento(id) |
| `producto_pkey` | PRIMARY KEY | producto | id_producto | — |
| `producto_cat_fk` | FOREIGN KEY | producto | id_producto_categoria | → producto_categoria(id) |
| `venta_pkey` | PRIMARY KEY | venta | id_venta | — |
| `tienda_fk` | FOREIGN KEY | venta | id_tienda | → tienda(id) |
| `cliente_fk` | FOREIGN KEY | venta | id_cliente | → cliente(id) |
| `vendedor_fk` | FOREIGN KEY | venta | id_vendedor | → vendedor(id) |
| `venta_detalle_pkey` | PRIMARY KEY | venta_detalle | id_venta_detalle | — |
| `venta_fk` | FOREIGN KEY | venta_detalle | id_venta | → venta(id) |
| `producto_fk` | FOREIGN KEY | venta_detalle | id_producto | → producto(id) |

---

## 5. Indices

| Indice | Tabla | Columnas | Tipo |
|--------|-------|----------|------|
| `pais_pkey` | pais | id_pais | UNIQUE B-tree |
| `idx_pais_lookup` | pais | id_pais, descripcion, territorio | B-tree |
| `producto_familia_pkey` | producto_familia | id_producto_familia | UNIQUE B-tree |
| `idx_producto_familia_lookup` | producto_familia | id, descripcion | B-tree |
| `tienda_pkey` | tienda | id_tienda | UNIQUE B-tree |
| `idx_tienda_lookup` | tienda | id, descripcion | B-tree |
| `ciudad_pkey` | ciudad | id_ciudad | UNIQUE B-tree |
| `idx_ciudad_lookup` | ciudad | id, nombre, departamento, id_pais | B-tree |
| `producto_departamento_pkey` | producto_departamento | id_producto_departamento | UNIQUE B-tree |
| `idx_producto_departamento_lookup` | producto_departamento | id, id_familia, descripcion | B-tree |
| `vendedor_pkey` | vendedor | id_vendedor | UNIQUE B-tree |
| `cliente_pkey` | cliente | id_cliente | UNIQUE B-tree |
| `idx_cliente_lookup` | cliente | id, id_ciudad, nombre, direccion | B-tree |
| `producto_categoria_pkey` | producto_categoria | id_producto_categoria | UNIQUE B-tree |
| `producto_pkey` | producto | id_producto | UNIQUE B-tree |
| `idx_producto_lookup` | producto | id, descripcion, id_categoria | B-tree |
| `venta_pkey` | venta | id_venta | UNIQUE B-tree |
| `idx_venta_lookup` | venta | id_venta | B-tree |
| `venta_detalle_pkey` | venta_detalle | id_venta_detalle | UNIQUE B-tree |
| `idx_venta_detalle_lookup` | venta_detalle | id_venta_detalle | B-tree |

---

## 6. Nota sobre tipos de PK

Las tablas base (`pais`, `producto_familia`, `tienda`, `producto_categoria`) usan `double precision` como PK. Las demas tablas usan `integer`. Las columnas FK en las tablas hijas son siempre `integer`, lo que genera un desajuste de tipos con las PKs de `double precision`. La compatibilidad se resuelve en la capa Java (JPA mapea `Double`/`Integer` segun corresponda).
