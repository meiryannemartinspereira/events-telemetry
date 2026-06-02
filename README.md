# Serviço de Eventos

## 📋 Visão Geral

O **Serviço de Eventos** é responsável por receber, processar e persistir eventos de telemetria e operação de veículos.

Os eventos são produzidos e enviados para tópicos no **Apache Kafka**, consumidos pelo serviço e armazenados no **PostgreSQL**, tornando-se a fonte de dados para consultas e dashboards.

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
┌─────────────┐
│ Middleware  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Dashboard   │
└─────────────┘
```

---

## 🔄 Fluxo de Processamento

1. Um **Producer** publica eventos em tópicos Kafka.
2. O **Consumer** consome os eventos.
3. Os eventos são persistidos no **PostgreSQL** (Source).
4. O **Middleware** consolida e enriquece os dados.
5. O **Dashboard** consulta os dados processados.
6. Serviços de cadastro mantêm atualizadas as informações de:
   - Motoristas
   - Veículos

---

## 📦 Modelo de Dados

### Driver

Representa o motorista responsável pelo veículo.

| Campo | Tipo | Descrição |
|---------|---------|---------|
| id | Integer | Identificador único |
| name | String | Nome do motorista |
| cpf | String | CPF do motorista |
| phone | String | Telefone |
| date_register | Timestamp | Data de cadastro |

---

### Vehicle

Representa um veículo vinculado a um motorista.

| Campo | Tipo | Descrição |
|---------|---------|---------|
| id | Integer | Identificador único |
| plate | String | Placa do veículo |
| model | String | Modelo |
| vehicle_year | Integer | Ano do veículo |
| driver_id | Integer | Referência ao motorista |

---

### Event

Representa um evento gerado por um veículo.

| Campo | Tipo | Descrição |
|---------|---------|---------|
| id | Integer | Identificador único |
| vehicle_id | Integer | Veículo responsável |
| type | String | Tipo do evento |
| description | String | Descrição do evento |
| location | String | Localização |
| date_time | Timestamp | Data e hora do evento |

---

## 📊 Relacionamento das Entidades

```text
Driver (1)
    │
    ▼
Vehicle (N)
    │
    ▼
Event (N)
```

### Regras de Negócio

- Um motorista pode possuir vários veículos.
- Um veículo pertence a apenas um motorista.
- Um veículo pode gerar vários eventos.
- Um evento pertence a apenas um veículo.

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

## 🚀 Endpoints

### Buscar todos os eventos

```http
GET /all/events
```

#### Exemplo de resposta

```json
[
  {
    "id": 1,
    "vehicle_id": 10,
    "type": "CHECKPOINT",
    "description": "Veículo chegou ao destino",
    "location": "Porto Alegre",
    "date_time": "2026-06-02T10:30:00"
  }
]
```

---

### Buscar evento por ID

```http
GET /event/{id}
```

#### Exemplo

```http
GET /event/1
```

---

### Buscar eventos por motorista

```http
GET /driver/{id}/events
```

#### Exemplo

```http
GET /driver/5/events
```

Retorna todos os eventos dos veículos vinculados ao motorista informado.

---

## 🎯 Tipos de Eventos

- CHECKPOINT
- CARGA
- DESCARGA
- ABASTECIMENTO
- MANUTENÇÃO
- INÍCIO DE VIAGEM
- FIM DE VIAGEM

---

## 🛠️ Tecnologias Utilizadas

- PostgreSQL
- Apache Kafka
- REST API
- Middleware de Integração
- Dashboard de Monitoramento

---

## 📈 Objetivo do Projeto

Disponibilizar uma plataforma centralizada para armazenamento, consulta e monitoramento de eventos operacionais de veículos, permitindo rastreabilidade completa desde a geração do evento até sua visualização no dashboard.
