#Blazor webassembly with .net 5/hostedmodel/individual authentication with docker compose

## certificate for windows docker
    - dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
    - dotnet dev-certs https --trust
        In the preceding commands, replace { password here } with a password.
        When using PowerShell, replace %USERPROFILE% with $env:USERPROFILE.
        Note: for other environment (https://docs.microsoft.com/en-us/aspnet/core/security/docker-https?view=aspnetcore-5.0)


#1. add appsettings.Production.json with
    "IdentityServer": {
    "Key": {
      "Type": "Store",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "Name": "CN=MyApplication"
    }

#2. Program.cs 
    public static async Task Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();

            using (var scope = host.Services.CreateScope())
            {
                var services = scope.ServiceProvider;

                try
                {
                    var context = services.GetRequiredService<ApplicationDbContext>();

                    if (context.Database.IsSqlServer())
                    {
                        context.Database.Migrate();
                    }

                    //var userManager = services.GetRequiredService<UserManager<ApplicationUser>>();
                    //var roleManager = services.GetRequiredService<RoleManager<IdentityRole>>();

                    //await ApplicationDbContextSeed.SeedDefaultUserAsync(userManager, roleManager);
                    //await ApplicationDbContextSeed.SeedSampleDataAsync(context);
                }
                catch (Exception ex)
                {
                    var logger = scope.ServiceProvider.GetRequiredService<ILogger<Program>>();

                    logger.LogError(ex, "An error occurred while migrating or seeding the database.");

                    throw;
                }
            }

            await host.RunAsync();
        }

#2. docker-compose.yml
    - clear docker-compose.override.yml
    - edit docker-compose.yml
        version: '3.4'

        services:
          blazorwasmhostedidentiy.server:
            image: ${DOCKER_REGISTRY-}blazorwasmhostedidentiyserver
            build:
              context: .
              dockerfile: BlazorWasmHostedIdentiy/Server/Dockerfile

            environment:
              - "ConnectionStrings__DefaultConnection=Server=db;Database=sampleDB;User=sa;Password=password;MultipleActiveResultSets=true"
              - "IdentityServer__Key__Type=Development"
              - "ASPNETCORE_Kestrel__Certificates__Default__Password=password"
              - "ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx"
            volumes:
              - ~/.aspnet/https:/https:ro
            ports:
              - "8080:80"
              - "8081:443"
            depends_on:
              - db
            restart: on-failure

          db:
            image: "mcr.microsoft.com/mssql/server"
            environment:
              - "SA_PASSWORD=password"
              - "ACCEPT_EULA=Y"
            ports:
              - "1433:1433"
            restart: on-failure

#3. startup.cs (configure SameSite and Secure options on Cookies generate by IS4)
    - https://github.com/IdentityServer/IdentityServer4/issues/4165
	- services.Configure<CookieAuthenticationOptions>(IdentityServerConstants.DefaultCookieAuthenticationScheme, options =>
            {
                options.Cookie.SameSite = SameSiteMode.None;
                options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
                options.Cookie.IsEssential = true;
            });

    - app.UseCookiePolicy(new CookiePolicyOptions
            {
                MinimumSameSitePolicy = SameSiteMode.None,
                Secure = CookieSecurePolicy.Always,
            });