# Serviço de Eventos

## 📋 Visão Geral

O Serviço de Eventos é responsável por receber, armazenar e disponibilizar eventos operacionais gerados pelos veículos da frota.

Os eventos são produzidos e enviados para tópicos no Apache Kafka. Um consumidor processa essas mensagens e persiste os dados no PostgreSQL, que atua como fonte oficial de consulta.

Além disso, serviços de cadastro mantêm atualizadas as informações de motoristas e veículos, permitindo o enriquecimento dos dados apresentados no Dashboard de Eventos.

---

## 🏗️ Arquitetura

```text
┌─────────────┐
│  Producer   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│    Kafka    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Consumer   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ PostgreSQL  │
│   (Source)  │
└──────┬──────┘
       │
       ▼
┌───────────────────┐
│   API de Eventos  │
└──────┬────────────┘
       │
       ▼
┌─────────────┐
│ Dashboard   │
└─────────────┘
```

---

## 🔄 Fluxo de Processamento

1. Um Producer publica eventos em tópicos Kafka.
2. O Consumer consome os eventos.
3. Os eventos são persistidos no PostgreSQL.
4. Serviços de cadastro atualizam as informações de motoristas e veículos.
5. A API de Eventos consulta e consolida os dados.
6. O Dashboard consome os endpoints disponibilizados pela API.

---

## 📦 Modelo de Dados

### Driver

Representa o motorista responsável pelo veículo.

| Campo | Tipo |
|---------|---------|
| id | Integer |
| name | String |
| cpf | String |
| phone | String |
| date_register | Timestamp |

---

### Vehicle

Representa um veículo vinculado a um motorista.

| Campo | Tipo |
|---------|---------|
| id | Integer |
| plate | String |
| model | String |
| vehicle_year | Integer |
| driver_id | Integer |

---

### Event

Representa um evento operacional gerado por um veículo.

| Campo | Tipo |
|---------|---------|
| id | Integer |
| vehicle_id | Integer |
| type | String |
| description | String |
| location | String |
| date_time | Timestamp |

---

## 📊 Diagrama Entidade Relacionamento

```text
+-------------+
|   driver    |
+-------------+
| id (PK)     |
| name        |
| cpf         |
| phone       |
| date_register
+-------------+
       |
       | 1:N
       ▼
+-------------+
|   vehicle   |
+-------------+
| id (PK)     |
| plate       |
| model       |
| vehicle_year|
| driver_id FK|
+-------------+
       |
       | 1:N
       ▼
+-------------+
|   events    |
+-------------+
| id (PK)     |
| vehicle_id FK
| type        |
| description |
| location    |
| date_time   |
+-------------+
```

---

## 📐 Regras de Negócio

- Um motorista pode possuir vários veículos.
- Um veículo pertence a apenas um motorista.
- Um veículo pode gerar vários eventos.
- Um evento pertence a apenas um veículo.
- Eventos não podem existir sem um veículo associado.
- Veículos não podem existir sem um motorista associado.

---

## 🗄️ Estrutura das Tabelas

### driver

```sql
CREATE TABLE driver (
    id SERIAL PRIMARY KEY,
    name VARCHAR(150) NOT NULL,
    cpf VARCHAR(14) UNIQUE NOT NULL,
    phone VARCHAR(20),
    date_register TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### vehicle

```sql
CREATE TABLE vehicle (
    id SERIAL PRIMARY KEY,
    plate VARCHAR(10) UNIQUE NOT NULL,
    model VARCHAR(100) NOT NULL,
    vehicle_year INTEGER,
    driver_id INTEGER NOT NULL REFERENCES driver(id)
);
```

### events

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    vehicle_id INTEGER NOT NULL REFERENCES vehicle(id),
    type VARCHAR(50) NOT NULL,
    description TEXT,
    location VARCHAR(255),
    date_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 📩 Exemplo de Evento Recebido

```json
{
  "vehicle_id": 15,
  "type": "CHECKPOINT",
  "description": "Chegada ao cliente",
  "location": "Porto Alegre - RS",
  "date_time": "2026-06-02T14:30:00"
}
```

---

## 🚀 API REST

### Listar todos os eventos

```http
GET /all/events
```

---

### Buscar evento por identificador

```http
GET /event/{id}
```

Exemplo:

```http
GET /event/1
```

---

### Buscar eventos por motorista

```http
GET /driver/{id}/events
```

Exemplo:

```http
GET /driver/5/events
```

Retorna todos os eventos dos veículos vinculados ao motorista informado.

---

## 🎯 Tipos de Eventos

| Tipo |
|--------|
| CHECKPOINT |
| CARGA |
| DESCARGA |
| ABASTECIMENTO |
| MANUTENÇÃO |
| INICIO_VIAGEM |
| FIM_VIAGEM |

---

## 🛠️ Stack Tecnológica

- Python
- FastAPI
- PostgreSQL
- Apache Kafka
- Docker
- REST API

---

## 📈 Objetivo

Disponibilizar uma API centralizada para consulta de eventos operacionais da frota, permitindo rastreabilidade completa desde a geração do evento até sua visualização em dashboards e sistemas consumidores.
