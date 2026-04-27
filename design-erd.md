# ERD Design, REST API, and MVC

Continue this after [express-mvc.md](./express-mvc.md).

This guide teaches you how to design a database using Entity Relationship Diagrams (ERDs), visualize it with dbdiagram.io, then build a REST API and MVC structure around that design.

## 1) What is an Entity Relationship Diagram (ERD)?

An ERD is a visual map of your database.

It shows:

1. **Entities:** Tables in your database (User, Post, Comment)
2. **Attributes:** Columns in each table (id, name, email)
3. **Relationships:** How tables connect to each other (one User has many Posts)
4. **Keys:** Primary keys (identify rows) and foreign keys (link tables)

Think of it like a blueprint before you build a house. You plan the rooms and hallways on paper first, not after.

Example:

```
A Player can own many Pokemon.
A Pokemon belongs to one Player.
A Pokemon is one species from Pokedex.
```

An ERD shows this visually.

## 2) Key ERD Concepts You Need

### Entity
An entity is a table. Example: `Player`, `Pokemon`, `Pokedex`.

### Attribute
An attribute is a column. Example: Player has `id`, `name`, `level`.

### Primary Key (PK)
A column that uniquely identifies a row. Every table has one.

Examples:
- `Player.id` is a PK
- `Pokemon.id` is a PK
- `Pokedex.number` is a PK

### Foreign Key (FK)
A column that links to another table's primary key.

Examples:
- `Pokemon.owner_id` is a FK that points to `Player.id`
- `Pokemon.dex_num` is a FK that points to `Pokedex.number`

### Relationships
How tables connect:

1. **One-to-Many:** One Player owns many Pokemon
2. **Many-to-One:** Many Pokemon belong to one Player
3. **One-to-One:** One Person has one Passport (less common in games)

## 3) Why dbdiagram.io is Better Than Manual Drawing

### Manual drawing (bad):

```
Pen and paper diagram:
    [Player]  ----  [Pokemon]  ----  [Pokedex]
```

**Problems:**

1. Hard to update (erase, redraw, messy)
2. Can't export or share easily
3. No SQL generation
4. Hard to see all details at once
5. Takes a lot of time to make it look nice

### dbdiagram.io (good):

1. **Instant SQL:** Click a button, get the SQL `CREATE TABLE` statements
2. **Drag-and-drop relationships:** Click and drag to create connections
3. **Auto-formatting:** Diagram stays clean and organized automatically
4. **Exportable:** Share as a link, download as SQL, PNG, or JSON
5. **Collaborative:** Share with your team in real-time
6. **Version history:** See past versions if you make mistakes
7. **Fast to update:** Change a field and the whole diagram updates instantly
8. **Readable:** Color-coded, clear visual relationships

**Bottom line:** dbdiagram.io turns your ERD into working SQL in seconds.

## 4) Step-by-Step: Build an ERD in dbdiagram.io

### Step 1: Go to dbdiagram.io

Open https://dbdiagram.io in your browser. No login required to start.

### Step 1.5: Understand the dbdiagram Interface

dbdiagram has three key features that make it powerful:

#### The DBML Editor (Left Panel)

- This is a code editor powered by Visual Studio Code
- You type your schema here using DBML syntax
- **Real-time preview:** As you type, the diagram updates instantly on the right

#### The Diagram Visualizer (Right Panel)

- Shows your tables, columns, and relationships visually
- Color-coded for easy reading
- **Drag and drop relationships:** Click a primary key and drag to a foreign key to create a connection

#### Keyboard Shortcuts (Power Tools)

- `Ctrl/Command + F12`: Jump from code to diagram (or vice versa)
- `Ctrl/Command + \`: Show/hide the editor to see more diagram space
- `Ctrl/Command + /`: Comment out a line
- `F1`: Open Command Palette for all available commands
- **Double-click on a table in the diagram:** Jump to its code definition
- **Double-click on code:** Jump to that element in the diagram

#### Navigation Between Code and Diagram

- **Code → Diagram:** Type code on the left, watch the diagram appear on the right
- **Diagram → Code:** Double-click a table/relationship on the diagram to jump to its code
- This two-way navigation keeps your schema and visualization in sync

#### View Notes

- Hover over table headers or columns in the diagram
- Click "Read Full" to see notes with markdown formatting
- Useful for documenting what each table or field means

### Step 2: Understand the left panel (SQL editor)

The left side has a text editor where you write your schema in `dbml` language (Database Markup Language).

It looks like:

```dbml
Table Player {
  id INT [pk, increment]
  name TEXT [not null]
}
```

The diagram updates instantly on the right.

### Step 3: Write your first table

Copy this into the left panel:

```dbml
Table Player {
  id INT [pk, increment]
  name TEXT [not null]
  level INT [not null]
  created_on timestamp
}
```

**What this means:**

- `id INT [pk, increment]`: ID is the primary key, auto-increment (1, 2, 3...)
- `name TEXT [not null]`: Name is text and required (cannot be empty)
- `level INT [not null]`: Level is a number and required
- `created_on timestamp`: When the player was created

Press Enter or click outside the editor, and a table appears on the right.

### Step 4: Add a second table

Add this below the Player table:

```dbml
Table Pokedex {
  number INT [pk]
  name TEXT [not null]
  type1 TEXT [not null]
  type2 TEXT
}
```

Now you see two separate tables on the diagram.

### Step 5: Add a third table with foreign keys

Add this below:

```dbml
Table Pokemon {
  id INT [pk, increment]
  owner_id INT [not null]
  dex_num INT [not null]
  hp INT
  atk INT
  def INT
}
```

**New syntax:**

- `owner_id INT [not null]`: Will link to Player.id (the owner of this Pokemon)
- `dex_num INT [not null]`: Will link to Pokedex.number (which species)

### Step 6: Draw relationships

Below all tables, add these lines:

```dbml
Ref: Pokemon.owner_id > Player.id
Ref: Pokemon.dex_num > Pokedex.number
```

**What this means:**

- `Ref:` = relationship
- `Pokemon.owner_id > Player.id`: Pokemon's owner_id points to a Player's id
- `Pokemon.dex_num > Pokedex.number`: Pokemon's dex_num points to a Pokedex number

Now lines appear on the diagram connecting the tables. You have a complete ERD.

### Step 7: Export your SQL

Click **Export > Export to SQL > LibSQL** at the top right.

dbdiagram.io generates standard SQL, but for modern projects you'll want to convert it to **Drizzle ORM** format. Here's the Drizzle ORM equivalent:

```typescript
import { integer, sqliteTable, text } from 'drizzle-orm/sqlite-core';
import { relations } from 'drizzle-orm';

// Tables
export const player = sqliteTable('player', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  name: text('name').notNull(),
  level: integer('level').notNull(),
  created_on: integer('created_at', { mode: 'timestamp' }).notNull().$defaultFn(() => new Date())
});

export const pokedex = sqliteTable('pokedex', {
  number: integer('number').primaryKey(),
  name: text('name').notNull(),
  type1: text('type1').notNull(),
  type2: text('type2'),
});

export const pokemon = sqliteTable('pokemon', {
  id: integer('id').primaryKey({ autoIncrement: true }),
  owner_id: integer('owner_id').notNull().references(() => player.id),
  dex_num: integer('dex_num').notNull().references(() => pokedex.number),
  hp: integer('hp'),
  atk: integer('atk'),
  def: integer('def'),
});

// Relationships
export const playerRelations = relations(player, ({ many }) => ({
  pokemon: many(pokemon),
}));

export const pokemonRelations = relations(pokemon, ({ one }) => ({
  player: one(player, {
    fields: [pokemon.owner_id],
    references: [player.id],
  }),
  pokedex: one(pokedex, {
    fields: [pokemon.dex_num],
    references: [pokedex.number],
  }),
}));

export const pokedexRelations = relations(pokedex, ({ many }) => ({
  pokemon: many(pokemon),
}));
```

**Why Drizzle ORM?**

- **Type-safe:** Your schema is TypeScript, so you catch errors at compile time
- **Relationships defined:** `.references()` creates foreign keys directly in the schema
- **Chainable API:** `.notNull().references()` reads like plain English
- **Migration-ready:** Use `drizzle-kit push` (or generate + migrate) to sync schema changes safely
- **Works with LibSQL:** Same database, better TypeScript experience

Save this as `src/schema.ts` in your project. You now have a type-safe database schema.

## 5) Complete Pokemon ERD Example

Here is the full dbml for the Pokemon game:

```dbml
Table Player {
  id INT [pk, increment]
  name TEXT [not null]
  level INT [not null]
  created_on timestamp
}

Table Pokedex {
  number INT [pk]
  name TEXT [not null]
  type1 TEXT [not null]
  type2 TEXT
}

Table Pokemon {
  id INT [pk, increment]
  owner_id INT [not null]
  dex_num INT [not null]
  hp INT
  atk INT
  def INT
}

Ref: Pokemon.owner_id > Player.id
Ref: Pokemon.dex_num > Pokedex.number
```

### Live Diagram Preview

Here is the Pokemon ERD visualized in dbdiagram.io. You can interact with it via this link directly: https://dbdiagram.io/d/Pokemon-643f4f646b31947051d2e8c7

**Try these interactions:**

1. **Hover over tables:** See column details
2. **Drag tables:** Reorganize the diagram layout
3. **Zoom in/out:** Use mouse wheel to see more detail
4. **Click relationships:** Highlight the FK and PK being connected
5. **Export:** Click the menu to export as SQL or PNG

### What this schema means:

1. **Player table:** Stores the player info. Each player has a unique `id`.
2. **Pokedex table:** A reference table. Lists all known Pokemon species (Pikachu, Bulbasaur, etc.)
3. **Pokemon table:** Stores the actual Pokemon instances a player owns.

### Example data:

```
Player table:
| id | name  | level | created_on |
|----|-------|-------|------------|
| 1  | Alice | 5     | 2026-01-01 |
| 2  | Bob   | 3     | 2026-01-02 |

Pokedex table:
| number | name      | type1    | type2    |
|--------|-----------|----------|----------|
| 25     | Pikachu   | Electric | (null)   |
| 1      | Bulbasaur | Grass    | Poison   |

Pokemon table (instances):
| id | owner_id | dex_num | hp | atk | def |
|----|----------|---------|----|----|-----|
| 1  | 1        | 25      | 35 | 55 | 40  |
| 2  | 1        | 1       | 45 | 49 | 49  |
| 3  | 2        | 25      | 35 | 55 | 40  |
```

Alice owns 2 Pokemon (Pikachu and Bulbasaur). Bob owns 1 Pokemon (Pikachu).

## 6) Transition to REST API Design

Once your ERD is done, you design REST API endpoints based on the tables.

### REST endpoint pattern:

```
GET /players          - get all players
GET /players/:id      - get one player
POST /players         - create a player
PUT /players/:id      - update a player
DELETE /players/:id   - delete a player

GET /pokemon          - get all pokemon instances
GET /pokemon/:id      - get one pokemon instance
POST /pokemon         - catch/create a pokemon
PUT /pokemon/:id      - update a pokemon (level up, rename)
DELETE /pokemon/:id   - release a pokemon

GET /pokedex          - get all pokemon species
GET /pokedex/:number  - get one species
```

### Why ERD → REST makes sense:

- Each table becomes a resource (`/players`, `/pokemon`, `/pokedex`)
- Each relationship becomes a nested route (e.g., `GET /players/1/pokemon` gets all Pokemon owned by player 1)

## 7) MVC Folder Structure for Pokemon API

Based on the ERD, your folders would look like:

```
project-root/
├─ index.js
├─ src/
│  ├─ routes/
│  │  ├─ playerRoutes.js
│  │  ├─ pokemonRoutes.js
│  │  └─ pokedexRoutes.js
│  ├─ controllers/
│  │  ├─ playerController.js
│  │  ├─ pokemonController.js
│  │  └─ pokedexController.js
│  └─ models/
│     ├─ playerModel.js
│     ├─ pokemonModel.js
│     └─ pokedexModel.js
├─ package.json
└─ .env
```

### Why this structure matches your ERD:

- **One folder per entity:** Each table gets its own controller, routes, and model
- **Separation:** Changes to Player logic don't touch Pokemon logic
- **Scalability:** Adding a new table is just adding new files
- **Testing:** Each model can be tested independently

### Example: playerController.js

```js
import { selectPlayers, selectPlayerById, insertPlayer } from '../models/playerModel.js';

export const getPlayers = (req, res) => {
  const result = selectPlayers();
  res.status(200).json(result);
  return;
};

export const getPlayerById = (req, res) => {
  const payload = { id: req.params.id };
  const result = selectPlayerById(payload);
  if (!result) {
    res.status(404).json({ message: 'Player not found' });
    return;
  }
  res.status(200).json(result);
  return;
};

export const postPlayer = (req, res) => {
  const { name, level } = req.body;
  if (!name || !level) {
    res.status(400).json({ message: 'name and level are required' });
    return;
  }
  const result = insertPlayer({ name, level });
  res.status(201).json(result);
  return;
};
```

### Example: playerModel.js

```js
const players = [];

export const selectPlayers = () => {
  return players;
};

export const selectPlayerById = (payload) => {
  return players.find(p => p.id === Number(payload.id)) ?? null;
};

export const insertPlayer = (payload) => {
  const result = {
    id: players.length + 1,
    name: payload.name,
    level: payload.level,
    created_on: new Date().toISOString()
  };
  players.push(result);
  return result;
};
```

## 8) How ERD, REST API, and MVC Connect

### The flow:

1. **ERD Design** (dbdiagram.io)
   - Plan your tables and relationships
   - Export SQL

2. **REST API Design**
   - One endpoint path per table
   - CRUD operations per table
   - Relationships become nested routes

3. **MVC Folder Structure**
   - One controller per table (handles HTTP)
   - One model per table (handles data/SQL)
   - One route file per table (maps endpoints)

### Real example: "Get all Pokemon owned by Player 1"

**ERD relationship:**
```
Player.id = 1 ──> Pokemon.owner_id = 1
```

**REST endpoint:**
```
GET /players/1/pokemon
```

**Controller logic:**
```js
export const getPokemonByPlayerId = (req, res) => {
  const payload = { owner_id: req.params.id };
  const result = selectPokemonByOwnerId(payload);
  res.status(200).json(result);
  return;
};
```

**Model logic:**
```js
const pokemon = [];

export const selectPokemonByOwnerId = (payload) => {
  return pokemon.filter(p => p.owner_id === Number(payload.owner_id));
};
```

The ERD relationship translates directly into the API design and code structure.

## 9) Student-Friendly Workflow

When you start a new project:

1. **Think about what data you need** (players, posts, comments, etc.)
2. **Sketch it on paper** (what fields? what relationships?)
3. **Go to dbdiagram.io** and build your ERD
4. **Export schema and sync with Drizzle**
5. **Design your REST endpoints** based on tables
6. **Create your MVC folder structure** (one controller/model per table)
7. **Write the code** (controller handles HTTP, model handles database)

This order is industry standard because:
- You avoid database redesigns mid-project
- Your API structure matches your data structure
- Your code organization is predictable

## 10) Final Thoughts

- **ERD design is 30% of the work, 70% saves you time later**
- **dbdiagram.io is free and saves hours** compared to manual SQL writing
- **REST API design becomes obvious once your ERD is clear**
- **MVC structure naturally follows your ERD** (one entity = one folder)

Start with dbdiagram.io. Every professional backend project begins here.
