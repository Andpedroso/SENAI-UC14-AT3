# Documentação ExoApi
### Desevolvimento de API Rest para o projeto ExoApi. A documentação apresenta funcionalidades e informações importantes para o desenvolvimento do sistema.
---
## Objetivo 🔮
### O desenvovlimento da API tem como objetivo a comunicação entre sistemas. Para a ExoApi será o acesso ao banco de dados com informações de projetos realizados pela empresa.
---
## Dificuldades e Soluções🔮
### A API desenvolvida possibilita a comunicação com uma hospedagem local. É necessário maiores configurações para a aplicação em um sistema de hospedagem paga.
* O arquivo launchSettings.json oferece informações de configuração.
```
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:33191",
      "sslPort": 44349
    }
  },
  "profiles": {
    "SENAI_UC14_AT2": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7149;http://localhost:5149",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```
### Podem ser aplicadas informações sobre os métodos para consulta.
* método BuscarPorId com o summary aplicado.
```
        /// <summary>
        /// Busca projeto por id.
        /// </summary>
        /// <param name="id">chave do campo.</param>
        /// <returns></returns>
        [HttpGet("{id}")]
        public IActionResult BuscarPorId(int id) 
        {
            try 
            {
                Projeto projeto = _projetoRepository.BuscarPorId(id);

                if(projeto == null) 
                {
                    return NotFound();
                }

                return Ok(projeto);
            }

            catch (Exception) 
            {
                throw;
            }
        }
```
### Mais tratamentos de erros podem ser aplicados e consultados.
* exemplo de tratamento de erro quando um id não existente é passado.
```
        [HttpPut("{id}")]
        public IActionResult Atualizar(int id, Projeto projeto)
        {
            try
            {
                Projeto projetoEncontrado = _projetoRepository.BuscarPorId(id);

                if (projetoEncontrado == null)
                {
                    return NotFound();
                }

                _projetoRepository.Atualizar(id, projeto);

                return StatusCode(204);
            }

            catch (Exception) 
            {
                throw;
            }
        }
```
* para soluções e consulta:

[Documentação API Web do ASP.NETCore](https://docs.microsoft.com/pt-br/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-6.0)

---
## Funcionamento 🔮
### O projeto apresenta basicamente 6 pastas e um arquivo cs para configurar a estrutura da API.
1. context
> Codificação para vincular a API com o banco.
>> ExoApiContext.cs
```
using Microsoft.EntityFrameworkCore;
using SENAI_UC14_AT2.Models;

namespace SENAI_UC14_AT2.Contexts
{
    public class ExoApiContext : DbContext
    {
        public ExoApiContext() { }

        public ExoApiContext(DbContextOptions<ExoApiContext> options) : base(options) { }

        protected override void
            OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            if (!optionsBuilder.IsConfigured)
            {
                optionsBuilder.UseSqlServer(
                    "Data Source=DESKTOP-Q4INLOT\\SQLEXPRESS;" +
                    "Initial Catalog=ExoApi;" +
                    "Integrated Security=False;" +
                    "User Id=sa;" +
                    "Password=server");
            }
        }

        public DbSet<Projeto> Projetos { get; set; }

        public DbSet<Usuario> Usuarios { get; set; }
    }
}

```
>>> esse é o bloco responsável pela configuração do banco:
```
optionsBuilder.UseSqlServer(
                    "Data Source=DESKTOP-Q4INLOT\\SQLEXPRESS;" +
                    "Initial Catalog=ExoApi;" +
                    "Integrated Security=False;" +
                    "User Id=sa;" +
                    "Password=server");
```
2. controllers
> Onde vão conter os controladores, que vão configurar a comunicação com o cliente. Vai conter o tipo de requisição nos métodos. Também poderão ser adicionadas regras de acesso nos métodos.
>> ProjetoController.cs
```
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using SENAI_UC14_AT2.Models;
using SENAI_UC14_AT2.Repositories;

namespace SENAI_UC14_AT2.Controllers
{
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    [Authorize]
    public class ProjetoController : ControllerBase
    {
        private readonly ProjetoRepository _projetoRepository;

        public ProjetoController(ProjetoRepository projetoRepository)
        {
            _projetoRepository = projetoRepository;
        }

        [HttpGet]
        public IActionResult Listar() 
        {
            try 
            {
                return Ok(_projetoRepository.Listar());
            }

            catch(Exception e) 
            {
                throw new Exception(e.Message);
            }
        }
        /// <summary>
        /// Busca projeto por id.
        /// </summary>
        /// <param name="id">chave do campo.</param>
        /// <returns></returns>
        [HttpGet("{id}")]
        public IActionResult BuscarPorId(int id) 
        {
            try 
            {
                Projeto projeto = _projetoRepository.BuscarPorId(id);

                if(projeto == null) 
                {
                    return NotFound();
                }

                return Ok(projeto);
            }

            catch (Exception) 
            {
                throw;
            }
        }

        [HttpPost]
        public IActionResult Cadastrar(Projeto projeto) 
        {
            try 
            {
                _projetoRepository.Cadastrar(projeto);

                return StatusCode(201);
            }

            catch (Exception) 
            {
                throw;
            }
        }

        [HttpPut("{id}")]
        public IActionResult Atualizar(int id, Projeto projeto)
        {
            try
            {
                Projeto projetoEncontrado = _projetoRepository.BuscarPorId(id);

                if (projetoEncontrado == null)
                {
                    return NotFound();
                }

                _projetoRepository.Atualizar(id, projeto);

                return StatusCode(204);
            }

            catch (Exception) 
            {
                throw;
            }
        }

        [Authorize(Roles = "1")]
        [HttpDelete("{id}")]
        public IActionResult Deletar(int id)
        {
            try
            {
                Projeto projetoEncontrado = _projetoRepository.BuscarPorId(id);

                if (projetoEncontrado == null)
                {
                    return NotFound();
                }

                _projetoRepository.Deletar(id);

                return StatusCode(204);
            }

            catch (Exception)
            {
                throw;
            }
        }
    }
}

```
>> UsuariosController.cs
````
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using SENAI_UC14_AT2.Interfaces;
using SENAI_UC14_AT2.Models;

namespace SENAI_UC14_AT2.Controllers
{
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    public class UsuariosController : ControllerBase
    {
        private readonly IUsuarioRepository _iUsuarioRepository;

        public UsuariosController(IUsuarioRepository usuarioRepository)
        {
            _iUsuarioRepository = usuarioRepository;
        }

        [HttpGet]
        public IActionResult Listar()
        {
            try
            {
                return Ok(_iUsuarioRepository.Listar());
            }

            catch (Exception)
            {
                throw;
            }
        }

        [HttpGet("{id}")]
        public IActionResult BuscarPorId(int id)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if(usuarioEncontrado == null)
                {
                    return NotFound();
                }

                return Ok(usuarioEncontrado);
            }

            catch (Exception)
            {
                throw;
            }
        }

        [HttpPost]
        public IActionResult Cadastrar(Usuario usuario)
        {
            try
            {
                _iUsuarioRepository.Cadastrar(usuario);

                return StatusCode(201);
            }

            catch (Exception)
            {
                throw;
            }
        }

        [HttpPut("{id}")]
        public IActionResult Alterar(int id, Usuario usuario)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if (usuarioEncontrado == null)
                {
                    return NotFound();
                }

                _iUsuarioRepository.Atualizar(id, usuario);

                return Ok("Usuario Alterado!");
            }

            catch (Exception)
            {
                throw;
            }
        }

        [HttpDelete("{id}")]
        public IActionResult Deletar(int id)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if (usuarioEncontrado == null)
                {
                    return NotFound();
                }

                _iUsuarioRepository.Deletar(id);

                return Ok("Usuario Deletado!");
            }

            catch (Exception)
            {
                throw;
            }
        }
    }
}
````
>> LoginController.cs
````
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using SENAI_UC14_AT2.Interfaces;
using SENAI_UC14_AT2.Models;
using SENAI_UC14_AT2.ViewModels;

namespace SENAI_UC14_AT2.Controllers
{
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    public class LoginController : ControllerBase
    {
        private readonly IUsuarioRepository _iUsuarioRepository;

        public LoginController(IUsuarioRepository iUsuarioRepository)
        {
            _iUsuarioRepository = iUsuarioRepository;
        }

        [HttpPost]
        public IActionResult Login(LoginViewModel login)
        {
            Usuario usuarioEncontrado = _iUsuarioRepository.Login(login.Email, login.Senha);

            if(usuarioEncontrado == null)
            {
                return Unauthorized(new { msg = "E-mail e/ou senha inválidos" });
            }

            var minhasClaims = new[]
            {
                new Claim(JwtRegisteredClaimNames.Email, usuarioEncontrado.Email),

                new Claim(JwtRegisteredClaimNames.Jti, usuarioEncontrado.Id.ToString()),

                new Claim(ClaimTypes.Role, usuarioEncontrado.Tipo)
            };

            var chave = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes("exoapi-chave-autenticacao"));

            var credenciais = new SigningCredentials(chave, SecurityAlgorithms.HmacSha256);

            var meuToken = new JwtSecurityToken
                (
                    issuer: "exoapi.webapi",

                    audience: "exoapi.webapi",

                    claims: minhasClaims,

                    expires: DateTime.Now.AddMinutes(60),

                    signingCredentials: credenciais

                );

            return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(meuToken) });
        }
    }
}
````
3. interfaces
> Definem a estrutura de uma classe, quando herdada, a classe deverá ter todos os métodos da interface.
>> IUsuarioRepository.cs
````
using SENAI_UC14_AT2.Models;

namespace SENAI_UC14_AT2.Interfaces
{
    public interface IUsuarioRepository
    {
        List<Usuario> Listar();

        Usuario BuscarPorId(int id);

        void Cadastrar(Usuario usuario);

        void Atualizar(int id, Usuario usuario);

        void Deletar(int id);

        Usuario Login(string email, string senha);
    }
}

````
4. models
> São os modelos que representam a entidade definindo seus atributos onde vão ser postos os valores.
>> Projeto.cs
````
namespace SENAI_UC14_AT2.Models
{
    public class Projeto
    {
        public int Id { get; set; }

        public string? Titulo { get; set; }

        public int StatusProjeto { get; set; }

        public string DataInicio { get; set; }

        public string Requisitos { get; set; }

        public string Area { get; set; }
    }
}
````
>> Usuario.cs
````
namespace SENAI_UC14_AT2.Models
{
    public class Usuario
    {
        public int Id { get; set; }

        public string? Email { get; set; }

        public string? Senha { get; set; }

        public string? Tipo { get; set; }
    }
}
````
5. repositories
> São configurações de acesso ao banco. Nele são definidos os métodos que vão possibiltar ações diretas no banco.
>> ProjetoRepository.cs
````
using SENAI_UC14_AT2.Contexts;
using SENAI_UC14_AT2.Models;

namespace SENAI_UC14_AT2.Repositories
{
    public class ProjetoRepository
    {
        private readonly ExoApiContext _context;

        public ProjetoRepository(ExoApiContext context)
        {
            _context = context;
        }

        public List<Projeto> Listar() 
        {
            return _context.Projetos.ToList();
        }

        public Projeto BuscarPorId(int id) 
        {
            return _context.Projetos.Find(id);
        }

        public void Cadastrar(Projeto projeto) 
        {
            _context.Projetos.Add(projeto);

            _context.SaveChanges();
        }

        public void Atualizar(int id, Projeto projeto)
        {
            Projeto projetoBuscado = _context.Projetos.Find(id);

            if(projetoBuscado != null)
            {
                projetoBuscado.Titulo = projeto.Titulo;

                projetoBuscado.StatusProjeto = projeto.StatusProjeto;

                projetoBuscado.DataInicio = projeto.DataInicio;

                projetoBuscado.Requisitos = projeto.Requisitos;

                projetoBuscado.Area = projeto.Area;

                _context.Projetos.Update(projetoBuscado);

                _context.SaveChanges();
            }
        }

        public void Deletar(int id)
        {
            Projeto projeto = _context.Projetos.Find(id);

            _context.Projetos.Remove(projeto);

            _context.SaveChanges();
        }
    }
}
````
>> UsuarioRepository.cs
````
using SENAI_UC14_AT2.Contexts;
using SENAI_UC14_AT2.Interfaces;
using SENAI_UC14_AT2.Models;

namespace SENAI_UC14_AT2.Repositories
{
    public class UsuarioRepository : IUsuarioRepository
    {
        private readonly ExoApiContext _context;

        public UsuarioRepository(ExoApiContext context)
        {
            _context = context;
        }

        public void Atualizar(int id, Usuario usuario)
        {
            Usuario usuarioEncontrado = _context.Usuarios.Find(id);

            if(usuarioEncontrado != null)
            {
                usuarioEncontrado.Email = usuario.Email;

                usuarioEncontrado.Senha = usuario.Senha;

                usuarioEncontrado.Tipo = usuario.Tipo;

                _context.Usuarios.Update(usuarioEncontrado);

                _context.SaveChanges();
            }
        }

        public Usuario BuscarPorId(int id)
        {
            return _context.Usuarios.Find(id);
        }

        public void Cadastrar(Usuario usuario)
        {
            _context.Usuarios.Add(usuario);

            _context.SaveChanges();
        }

        public void Deletar(int id)
        {
            Usuario usuarioEncontrado = _context.Usuarios.Find(id);

            _context.Usuarios.Remove(usuarioEncontrado);

            _context.SaveChanges();
        }

        public List<Usuario> Listar()
        {
            return _context.Usuarios.ToList();
        }

        public Usuario Login(string email, string senha)
        {
            return _context.Usuarios.FirstOrDefault(x => x.Email == email && x.Senha == senha);
        }
    }
}
````
6. viewModels
> Permite a configuração de um modelo para sua view. No projeto foi criado para o login.
>> LoginViewModel.cs
````
using System.ComponentModel.DataAnnotations;

namespace SENAI_UC14_AT2.ViewModels
{
    public class LoginViewModel
    {
        [Required(ErrorMessage = "E-mail Requerido")]
        public string Email { get; set; }

        [Required(ErrorMessage ="Senha Requerida")]
        public string Senha { get; set; }
    }
}
````
7. Program.cs
> Contém configurações de inicialização da API.
>> Program.cs
````
using Microsoft.IdentityModel.Tokens;
using SENAI_UC14_AT2.Contexts;
using SENAI_UC14_AT2.Interfaces;
using SENAI_UC14_AT2.Repositories;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();

builder.Services.AddCors(options =>
{
    options.AddPolicy("CorsPolicy", builder =>
    {
        builder.WithOrigins("http://localhost:3000")
        .AllowAnyHeader()
        .AllowAnyMethod();
    });
});

builder.Services.AddAuthentication(options =>
{
    options.DefaultChallengeScheme = "JwtBearer";

    options.DefaultAuthenticateScheme = "JwtBearer";

}).AddJwtBearer("JwtBearer", options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = true,

        ValidateAudience = true,

        ValidateLifetime = true,

        IssuerSigningKey = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes("exoapi-chave-autenticacao")),

        ClockSkew = TimeSpan.FromMinutes(60),

        ValidIssuer = "exoapi.webapi",

        ValidAudience = "exoapi.webapi"
    };
});

builder.Services.AddScoped<ExoApiContext, ExoApiContext>();

builder.Services.AddTransient<ProjetoRepository, ProjetoRepository>();

builder.Services.AddTransient<IUsuarioRepository, UsuarioRepository>();

// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle

builder.Services.AddEndpointsApiExplorer();

builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline.

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseCors("CorsPolicy");

app.UseAuthentication();

app.UseAuthorization();

app.MapControllers();

app.Run();
````
---
## Funcionalidades e recursos 🔮
### A seguir serão apresentadas as funcionalidades e recursos.
1. Usuários
> 1. Cadastrar usuário
>> Método no Controller 
````
        [HttpPost]
        public IActionResult Cadastrar(Usuario usuario)
        {
            try
            {
                _iUsuarioRepository.Cadastrar(usuario);

                return StatusCode(201);
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Cadastrar(Usuario usuario)
        {
            _context.Usuarios.Add(usuario);

            _context.SaveChanges();
        }
````
>> JSON
````
> Swagger

{
  "email": "email3@sp.br",
  "senha": "12345",
  "tipo": "1"
}

>> response de sucesso:

content-length: 0 
date: Tue,31 May 2022 22:24:08 GMT 
server: Kestrel
````
> 2. Alterar Usuário
>> Método no Controller
````
        [HttpPut("{id}")]
        public IActionResult Alterar(int id, Usuario usuario)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if (usuarioEncontrado == null)
                {
                    return NotFound();
                }

                _iUsuarioRepository.Atualizar(id, usuario);

                return Ok("Usuario Alterado!");
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Atualizar(int id, Usuario usuario)
        {
            Usuario usuarioEncontrado = _context.Usuarios.Find(id);

            if(usuarioEncontrado != null)
            {
                usuarioEncontrado.Email = usuario.Email;

                usuarioEncontrado.Senha = usuario.Senha;

                usuarioEncontrado.Tipo = usuario.Tipo;

                _context.Usuarios.Update(usuarioEncontrado);

                _context.SaveChanges();
            }
        }
````
>> JSON
````
> Swagger

{
  "email": "email3@sp.br",
  "senha": "123456",
  "tipo": "1"
}

>> response de sucesso:

"Usuario Alterado!"
````
> 3. Ler usuários
>> Método no Controller
````
        [HttpGet]
        public IActionResult Listar()
        {
            try
            {
                return Ok(_iUsuarioRepository.Listar());
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public List<Usuario> Listar()
        {
            return _context.Usuarios.ToList();
        }
````
>> JSON
````
> Swagger

[
  {
    "id": 1,
    "email": "email@sp.br",
    "senha": "1234",
    "tipo": "0"
  },
  {
    "id": 7,
    "email": "email2@sp.br",
    "senha": "1234",
    "tipo": "1"
  },
  {
    "id": 8,
    "email": "email3@sp.br",
    "senha": "123456",
    "tipo": "1"
  }
]

>> response de sucesso:

 content-type: application/json; charset=utf-8 
 date: Tue,31 May 2022 22:32:38 GMT 
 server: Kestrel
````
> 4. Buscar usuário por id
>> Método no Controller
````
        [HttpGet("{id}")]
        public IActionResult BuscarPorId(int id)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if(usuarioEncontrado == null)
                {
                    return NotFound();
                }

                return Ok(usuarioEncontrado);
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public Usuario BuscarPorId(int id)
        {
            return _context.Usuarios.Find(id);
        }
````
>> JSON
````
> Swagger

{
  "id": 8,
  "email": "email3@sp.br",
  "senha": "123456",
  "tipo": "1"
}

>> response de sucesso:

 content-type: application/json; charset=utf-8 
 date: Tue,31 May 2022 22:34:04 GMT 
 server: Kestrel 
````
> 5. Deletar Usuário
>> Método no Controller
````
        [HttpDelete("{id}")]
        public IActionResult Deletar(int id)
        {
            try
            {
                Usuario usuarioEncontrado = _iUsuarioRepository.BuscarPorId(id);

                if (usuarioEncontrado == null)
                {
                    return NotFound();
                }

                _iUsuarioRepository.Deletar(id);

                return Ok("Usuario Deletado!");
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Deletar(int id)
        {
            Usuario usuarioEncontrado = _context.Usuarios.Find(id);

            _context.Usuarios.Remove(usuarioEncontrado);

            _context.SaveChanges();
        }
````
>> JSON
````
> Swagger

>> response de sucesso:

"Usuario Deletado!"
````
2. Projetos
> 1. Cadastrar projeto
>> Método no Controller 
````
        [HttpPost]
        public IActionResult Cadastrar(Projeto projeto) 
        {
            try 
            {
                _projetoRepository.Cadastrar(projeto);

                return StatusCode(201);
            }

            catch (Exception) 
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Cadastrar(Projeto projeto) 
        {
            _context.Projetos.Add(projeto);

            _context.SaveChanges();
        }
````
>> JSON
````
> Swagger

{
  "titulo": "AgroTest",
  "statusProjeto": 10,
  "dataInicio": "31/05/2022",
  "requisitos": "1.CRUD completo.",
  "area": "Agronomia"
}

>> response de sucesso

 content-length: 0 
 date: Tue,31 May 2022 22:43:22 GMT 
 server: Kestrel 
````
> 2. Alterar projeto
>> Método no Controller
````
        [HttpPut("{id}")]
        public IActionResult Atualizar(int id, Projeto projeto)
        {
            try
            {
                Projeto projetoEncontrado = _projetoRepository.BuscarPorId(id);

                if (projetoEncontrado == null)
                {
                    return NotFound();
                }

                _projetoRepository.Atualizar(id, projeto);

                return StatusCode(204);
            }

            catch (Exception) 
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Atualizar(int id, Projeto projeto)
        {
            Projeto projetoBuscado = _context.Projetos.Find(id);

            if(projetoBuscado != null)
            {
                projetoBuscado.Titulo = projeto.Titulo;

                projetoBuscado.StatusProjeto = projeto.StatusProjeto;

                projetoBuscado.DataInicio = projeto.DataInicio;

                projetoBuscado.Requisitos = projeto.Requisitos;

                projetoBuscado.Area = projeto.Area;

                _context.Projetos.Update(projetoBuscado);

                _context.SaveChanges();
            }
        }
````
>> JSON
````
> Swagger

{
  "titulo": "AgroTest",
  "statusProjeto": 60,
  "dataInicio": "31/05/2022",
  "requisitos": "1.CRUD completo.",
  "area": "Agronomia"
}

>> response de sucesso

 date: Tue,31 May 2022 22:44:56 GMT 
 server: Kestrel
````
> 3. Ler projeto.
>> Método no Controller
````
        [HttpGet]
        public IActionResult Listar() 
        {
            try 
            {
                return Ok(_projetoRepository.Listar());
            }

            catch(Exception e) 
            {
                throw new Exception(e.Message);
            }
        }
````
>> Método no Repository
````
        public List<Projeto> Listar() 
        {
            return _context.Projetos.ToList();
        }
````
>> JSON
````
> Swagger

[
  {
    "id": 6,
    "titulo": "DataCenter2",
    "statusProjeto": 100,
    "dataInicio": "22/05/2022",
    "requisitos": "1.Guardar clientes(Nome, Idade, CPF e E-mail), 2.CRUD Completo.",
    "area": "Vendas"
  },
  {
    "id": 7,
    "titulo": "AgroTest",
    "statusProjeto": 60,
    "dataInicio": "31/05/2022",
    "requisitos": "1.CRUD completo.",
    "area": "Agronomia"
  }
]

>> response de sucesso

 content-type: application/json; charset=utf-8 
 date: Tue,31 May 2022 22:46:15 GMT 
 server: Kestrel 
````
> 4. Buscar projeto por id
>> Método no Controller
````
        [HttpGet("{id}")]
        public IActionResult BuscarPorId(int id) 
        {
            try 
            {
                Projeto projeto = _projetoRepository.BuscarPorId(id);

                if(projeto == null) 
                {
                    return NotFound();
                }

                return Ok(projeto);
            }

            catch (Exception) 
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public Projeto BuscarPorId(int id) 
        {
            return _context.Projetos.Find(id);
        }
````
>> JSON
````
> Swagger

{
  "id": 7,
  "titulo": "AgroTest",
  "statusProjeto": 60,
  "dataInicio": "31/05/2022",
  "requisitos": "1.CRUD completo.",
  "area": "Agronomia"
}

>> response de sucesso

 content-type: application/json; charset=utf-8 
 date: Tue,31 May 2022 22:47:24 GMT 
 server: Kestrel 
````
> 5. Deletar Projeto
>> Método no Controller
````
        [Authorize(Roles = "1")]
        [HttpDelete("{id}")]
        public IActionResult Deletar(int id)
        {
            try
            {
                Projeto projetoEncontrado = _projetoRepository.BuscarPorId(id);

                if (projetoEncontrado == null)
                {
                    return NotFound();
                }

                _projetoRepository.Deletar(id);

                return StatusCode(204);
            }

            catch (Exception)
            {
                throw;
            }
        }
````
>> Método no Repository
````
        public void Deletar(int id)
        {
            Projeto projeto = _context.Projetos.Find(id);

            _context.Projetos.Remove(projeto);

            _context.SaveChanges();
        }
````
>> JSON
````
> Swagger

>> response de sucesso
 date: Tue,31 May 2022 22:48:14 GMT 
 server: Kestrel
````
4. Login e segurança
> 1. Configurações do método de geração de token
>> 1. Controller
````
        [HttpPost]
        public IActionResult Login(LoginViewModel login)
        {
            Usuario usuarioEncontrado = _iUsuarioRepository.Login(login.Email, login.Senha);

            if(usuarioEncontrado == null)
            {
                return Unauthorized(new { msg = "E-mail e/ou senha inválidos" });
            }

            var minhasClaims = new[]
            {
                new Claim(JwtRegisteredClaimNames.Email, usuarioEncontrado.Email),

                new Claim(JwtRegisteredClaimNames.Jti, usuarioEncontrado.Id.ToString()),

                new Claim(ClaimTypes.Role, usuarioEncontrado.Tipo)
            };

            var chave = new SymmetricSecurityKey(System.Text.Encoding.UTF8.GetBytes("exoapi-chave-autenticacao"));

            var credenciais = new SigningCredentials(chave, SecurityAlgorithms.HmacSha256);

            var meuToken = new JwtSecurityToken
                (
                    issuer: "exoapi.webapi",

                    audience: "exoapi.webapi",

                    claims: minhasClaims,

                    expires: DateTime.Now.AddMinutes(60),

                    signingCredentials: credenciais

                );

            return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(meuToken) });
        }
````
>> 2. Repository
````
        public Usuario Login(string email, string senha)
        {
            return _context.Usuarios.FirstOrDefault(x => x.Email == email && x.Senha == senha);
        }
````
>> 3. ViewModel
````
    public class LoginViewModel
    {
        [Required(ErrorMessage = "E-mail Requerido")]
        public string Email { get; set; }

        [Required(ErrorMessage ="Senha Requerida")]
        public string Senha { get; set; }
    }
````
>> JSON
````
> Swagger

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImVtYWlsQHNwLmJyIiwianRpIjoiMSIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6IjAiLCJleHAiOjE2NTQwNDAyMTQsImlzcyI6ImV4b2FwaS53ZWJhcGkiLCJhdWQiOiJleG9hcGkud2ViYXBpIn0.uldSQNZPWzgIJqAarSCU75FGHnc7VhvFTB2K8LQyMR0"
}

>> response de sucesso:

 content-type: application/json; charset=utf-8 
 date: Tue,31 May 2022 22:36:54 GMT 
 server: Kestrel
````
>> 4. Aplicação das restrições. 
* O [Authorize] poderá ser posto em cima de qualquer método ou classe. Roles define o tipo de usuário que pode ser autorizado. Caso tenha apenas o [Authorize] todos os usuários autenticados poderão acessar.
````
[Authorize(Roles = "1")]
[HttpDelete("{id}")]
````
---
## Melhorias futuras 🔮
### A seguir serão apresentadas as possíveis melhorias para o projeto.
1. Poderão ser configuradas novas autorizações.
2. Poderão ser configuradas novas mensagens de tratamento de erros.
3. Poderão ser definidas novas informações no summary dos métodos.