# MovieLand

MovieLand is a movie web shop built with ASP.NET Core Razor Pages, originally created as an e-commerce project for a university class. Visitors can browse and filter a seeded movie catalog, and registered users can review movies, keep favorites, compare titles side by side, fill a shopping cart and place orders. Admins manage the catalog (movies, directors and genres) from a dedicated admin area.

The solution follows a layered (clean architecture) design: a Domain core, an Application layer with services and DTOs, an Infrastructure layer with Entity Framework Core repositories, and a Razor Pages Web front end.

## Features

- **Catalog browsing** – search movies by title and filter by decade, director, genre, price range or rating.
- **Movie details** – per-movie pages (addressed by slug) with user reviews; signed-in users can add and delete reviews.
- **Favorites and compare** – save movies to a favorites list or compare several movies side by side.
- **Cart and orders** – add movies to a cart, check out with cash or card, and view, edit or cancel orders.
- **Accounts and roles** – registration and login via ASP.NET Core Identity, with `SuperAdmin`, `Admin` and `Client` roles.
- **Admin area** – create, edit and delete movies, directors and genres (restricted to `Admin` and `SuperAdmin`).
- **Contact form** – sends messages over SMTP.
- **Automatic database setup** – migrations are applied and seed data is inserted on startup.

## Tech stack

- .NET 5 / ASP.NET Core Razor Pages
- Entity Framework Core 5 with SQL Server
- ASP.NET Core Identity
- AutoMapper
- Bootstrap, jQuery and jQuery Validation (bundled in `wwwroot/lib`)
- SCSS compiled via BuildWebCompiler / SharpScss

## Prerequisites

- [.NET 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
- SQL Server (SQL Server Express or LocalDB is fine for local development)
- Optionally Visual Studio 2019 (16.8+) or later, which can open `MovieLand-WebShop.sln` directly

## Getting started

1. Clone the repository:

   ```bash
   git clone https://github.com/nkelava/movieland.git
   cd movieland
   ```

2. Point the app at your SQL Server instance by editing the connection string in `src/MovieLand.Web/appsettings.json` (see [Configuration](#configuration)).

3. Restore and run the web project:

   ```bash
   dotnet restore
   dotnet run --project src/MovieLand.Web
   ```

4. Open https://localhost:5001 (or http://localhost:5000).

On first start the app applies the EF Core migrations, creates the database and seeds roles, a super admin account, directors, genres and movies. There is no need to run `dotnet ef database update` manually.

### Default admin account

The seed creates a super admin you can use to reach the admin pages:

| Email                      | Password        |
| -------------------------- | --------------- |
| `superadmin@movieland.com` | `superadmin123` |

Change or remove this account before deploying anywhere public.

## Configuration

Settings live in `src/MovieLand.Web/appsettings.json` (with overrides in `appsettings.Development.json`). The only value you normally need to change is the connection string:

```json
{
  "ConnectionStrings": {
    "MovieLandConnection": "Server=.\\SQLEXPRESS;Database=MovieShop;Trusted_Connection=True;"
  }
}
```

For example, to use LocalDB:

```text
Server=(localdb)\\mssqllocaldb;Database=MovieShop;Trusted_Connection=True;
```

Like any ASP.NET Core setting, it can also be supplied through an environment variable instead of editing the file:

```bash
export ConnectionStrings__MovieLandConnection="Server=localhost;Database=MovieShop;User Id=sa;Password=<your-password>;"
```

| Variable                                | Purpose                                                               |
| --------------------------------------- | --------------------------------------------------------------------- |
| `ConnectionStrings__MovieLandConnection` | SQL Server connection string (overrides `appsettings.json`)           |
| `ASPNETCORE_ENVIRONMENT`                | `Development` (set by `launchSettings.json`) enables detailed errors  |

The contact form's SMTP settings (a Mailtrap inbox) are currently hard-coded in `src/MovieLand.Infrastructure/Services/EmailService.cs`; edit that file to use your own mail server.

## Database migrations

Migrations are stored in the Web project (`src/MovieLand.Web/Migrations`). To add a new one after changing the model, install the EF Core tools and run:

```bash
dotnet tool install --global dotnet-ef --version 5.0.9
dotnet ef migrations add <MigrationName> --project src/MovieLand.Web
```

The new migration is applied automatically the next time the app starts.

## Project structure

```text
movieland/
├── MovieLand-WebShop.sln          # Solution file
└── src/
    ├── MovieLand.Domain/          # Core entities, enums, repository interfaces, specifications
    ├── MovieLand.Application/     # Services, DTOs, AutoMapper profiles, service interfaces
    ├── MovieLand.Infrastructure/  # EF Core DbContext, seed data, repositories, email service, logging
    └── MovieLand.Web/             # ASP.NET Core Razor Pages app (entry point)
        ├── Pages/
        │   ├── Account/           # Login and registration
        │   ├── Admin/             # Movie, director and genre management
        │   ├── Movies/            # Movie detail page and reviews
        │   ├── Cart/  Order/      # Shopping cart and order pages
        │   ├── Favorites/  Compare/
        │   ├── AboutUs/  ContactUs/
        │   └── Shared/            # Layout, header, footer and partials
        ├── Migrations/            # EF Core migrations
        ├── wwwroot/               # Static assets: css, scss, js, images, client libraries
        ├── appsettings.json       # Configuration (connection string, logging)
        ├── Program.cs             # Host setup and database seeding
        └── Startup.cs             # Dependency injection, Identity and request pipeline
```

Dependencies flow inward: `Web` → `Infrastructure` → `Application` → `Domain`.
