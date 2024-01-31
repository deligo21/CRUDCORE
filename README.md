/////////////////////////////////////CRUD/////////////////////////////////////////

Procedimientos almacenados
```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE SP_Listar

AS
BEGIN

	SET NOCOUNT ON;

	SELECT * from Contacto
END
GO
```
```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE SP_Obtener
(@IdContacto int)
AS
BEGIN

	SET NOCOUNT ON;

	SELECT * from Contacto where IdContacto = @IdContacto
END
GO
```
```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE SP_Guardar
(@Nombre nvarchar(50),
@Telefono varchar(50),
@Correo nvarchar(50))
AS
BEGIN

	SET NOCOUNT ON;

	INSERT INTO Contacto (Nombre, Telefono, Correo) values (@Nombre, @Telefono, @Correo)
END
GO
```
```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE SP_Editar
(@IdContacto int,
@Nombre nvarchar(50),
@Telefono varchar(50),
@Correo nvarchar(50))
AS
BEGIN

	SET NOCOUNT ON;

	Update Contacto set Nombre = @Nombre, Telefono = @Telefono, Correo = @Correo where IdContacto = @IdContacto
END
GO
```
```
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
CREATE PROCEDURE SP_Eliminar
(@IdContacto int)
AS
BEGIN

	SET NOCOUNT ON;

	Delete from Contacto where @IdContacto = IdContacto
END
GO
```

Modelo
```
using System.ComponentModel.DataAnnotations;

namespace CRUDCORE.Models
{
    public class ContactoModel
    {
        public int IdContacto { get; set; }
        [Required (ErrorMessage ="El campo {0} es obligatorio")]
        public string Nombre { get; set; }
        [Required (ErrorMessage = "El campo {0} es obligatorio")]
        public string Telefono { get; set; }
        [Required (ErrorMessage = "El campo {0} es obligatorio")]
        public string Correo { get; set; }
    }
}
```
AppSettingsDevelopment
```
"ConnectionStrings": {
    "DefaultConnection": "Server=DELIGO21-PC;Database=DBCRUDCORE;Integrated Security=True;TrustServerCertificate=True"
  },
```
Crear carpeta Datos y luego archivo conexion
```
using System.Data.SqlClient;

namespace CRUDCORE.Datos
{
    public class Conexion
    {
        private string cadenaSQL = string.Empty;
        public Conexion() { 
            var builder = new ConfigurationBuilder().SetBasePath(Directory.GetCurrentDirectory()).AddJsonFile("appsettings.Development.json").Build();
            cadenaSQL = builder.GetSection("ConnectionStrings:DefaultConnection").Value;
        }

        public string GetCadenaSQL()
        {
            return cadenaSQL;
        }
    }
}
```
Luego crear en la misma carpeta ContactoDatos
```
using CRUDCORE.Models;
using System.Data.SqlClient;
using System.Data;
using static System.Runtime.InteropServices.JavaScript.JSType;

namespace CRUDCORE.Datos
{
    public class ContactoDatos
    {
        public List<ContactoModel> Listar()
        {
            var oLista = new List<ContactoModel>();
            var cn = new Conexion();

            using (var conexion = new SqlConnection(cn.GetCadenaSQL()))
            {
                conexion.Open();
                SqlCommand cmd = new SqlCommand("SP_Listar", conexion);
                cmd.CommandType = CommandType.StoredProcedure;

                using (var dr = cmd.ExecuteReader())
                {
                    while(dr.Read())
                    {
                        oLista.Add(new ContactoModel
                        {
                            IdContacto = Convert.ToInt32(dr["IdContacto"]),
                            Nombre = dr["Nombre"].ToString(),
                            Telefono = dr["Telefono"].ToString(),
                            Correo = dr["Correo"].ToString(),
                        });
                    }
                }
            }
            return oLista;
        }

        public ContactoModel Obtener(int IdContacto)
        {
            var oContacto = new ContactoModel();
            var cn = new Conexion();

            using (var conexion = new SqlConnection(cn.GetCadenaSQL()))
            {
                conexion.Open();
                SqlCommand cmd = new SqlCommand("SP_Obtener", conexion);
                cmd.Parameters.AddWithValue("IdContacto", IdContacto);
                cmd.CommandType = CommandType.StoredProcedure;

                using (var dr = cmd.ExecuteReader())
                {
                    while (dr.Read())
                    {
                        oContacto.IdContacto = Convert.ToInt32(dr["IdContacto"]);
                        oContacto.Nombre = dr["Nombre"].ToString();
                        oContacto.Telefono = dr["Telefono"].ToString();
                        oContacto.Correo = dr["Correo"].ToString();
                    }
                }
            }
            return oContacto;
        }

        public bool Guardar(ContactoModel oContacto)
        {
            bool rpta;

            try
            {
                var cn = new Conexion();

                using (var conexion = new SqlConnection(cn.GetCadenaSQL()))
                {
                    conexion.Open();
                    SqlCommand cmd = new SqlCommand("SP_Guardar", conexion);
                    cmd.Parameters.AddWithValue("Nombre", oContacto.Nombre);
                    cmd.Parameters.AddWithValue("Telefono", oContacto.Telefono);
                    cmd.Parameters.AddWithValue("Correo", oContacto.Correo);
                    cmd.CommandType = CommandType.StoredProcedure;
                    cmd.ExecuteNonQuery();

                }
                rpta = true;
            }
            catch (Exception ex)
            {
                string error = ex.Message;
                rpta=false;
            }

            return rpta;
        }

        public bool Editar(ContactoModel oContacto)
        {
            bool rpta;

            try
            {
                var cn = new Conexion();

                using (var conexion = new SqlConnection(cn.GetCadenaSQL()))
                {
                    conexion.Open();
                    SqlCommand cmd = new SqlCommand("SP_Editar", conexion);
                    cmd.Parameters.AddWithValue("IdContacto", oContacto.IdContacto);
                    cmd.Parameters.AddWithValue("Nombre", oContacto.Nombre);
                    cmd.Parameters.AddWithValue("Telefono", oContacto.Telefono);
                    cmd.Parameters.AddWithValue("Correo", oContacto.Correo);
                    cmd.CommandType = CommandType.StoredProcedure;
                    cmd.ExecuteNonQuery();

                }
                rpta = true;
            }
            catch (Exception ex)
            {
                string error = ex.Message;
                rpta = false;
            }

            return rpta;
        }

        public bool Eliminar(int IdContacto)
        {
            bool rpta;

            try
            {
                var cn = new Conexion();

                using (var conexion = new SqlConnection(cn.GetCadenaSQL()))
                {
                    conexion.Open();
                    SqlCommand cmd = new SqlCommand("SP_Eliminar", conexion);
                    cmd.Parameters.AddWithValue("IdContacto", IdContacto);
                    cmd.CommandType = CommandType.StoredProcedure;
                    cmd.ExecuteNonQuery();

                }
                rpta = true;
            }
            catch (Exception ex)
            {
                string error = ex.Message;
                rpta = false;
            }

            return rpta;
        }
    }
}
```
Luego crear Mantenedor Controller
```
using Microsoft.AspNetCore.Mvc;
using CRUDCORE.Datos;
using CRUDCORE.Models;

namespace CRUDCORE.Controllers
{
    public class MantenedorController : Controller
    {

        ContactoDatos _ContactoDatos = new ContactoDatos();

        public IActionResult Listar()
        {
            var oLista = _ContactoDatos.Listar();
            return View(oLista);
        }

        public IActionResult Guardar()
        {
            //Get
            return View();
        }

        [HttpPost]
        public IActionResult Guardar(ContactoModel oContacto)
        {
            //Post
            if (!ModelState.IsValid)
            {
                return View();
            }
            var respuesta = _ContactoDatos.Guardar(oContacto);
            if (respuesta)
                return RedirectToAction("Listar");
            else
                return View();
        }

        public IActionResult Editar(int IdContacto)
        {
            var oContacto = _ContactoDatos.Obtener(IdContacto);
            return View(oContacto);
        }

        [HttpPost]
        public IActionResult Editar(ContactoModel oContacto)
        {
            //Post
            if (!ModelState.IsValid)
            {
                return View();
            }
            var respuesta = _ContactoDatos.Editar(oContacto);
            if (respuesta)
                return RedirectToAction("Listar");
            else
                return View();
        }
	public IActionResult Eliminar(int IdContacto)
{
    var oContacto = _ContactoDatos.Obtener(IdContacto);
    return View(oContacto);
}

[HttpPost]
public IActionResult Eliminar(ContactoModel oContacto)
{

    var respuesta = _ContactoDatos.Eliminar(oContacto.IdContacto);
    if (respuesta)
        return RedirectToAction("Listar");
    else
        return View();
}
    }
	
}
```
Luego crear una vista por cada metodo del controller
Instalar RunTimeCompilation
Agregar la siguiente linea al program cs
```
builder.Services.AddRazorPages().AddRazorRuntimeCompilation();
```
En la Vista de Guardar agregar el siguiente codigo:
```
@model ContactoModel
@{
    ViewData["Title"] = "Guardar";
    Layout = "~/Views/Shared/_Layout.cshtml";
}


<div class="card">
    <div class="card-header">
        Crear Contacto
    </div>
    <div class="card-body">
        <form asp-action="Guardar" asp-controller="Mantenedor" method="post">
            <div class="mb-3">
                <label class="form-label">Nombre</label>
                <input asp-for="Nombre" type="text" class="form-control">
                <span asp-validation-for="Nombre" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label class="form-label">Telefono</label>
                <input asp-for="Telefono" type="text" class="form-control">
                <span asp-validation-for="Telefono" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label class="form-label">Correo</label>
                <input asp-for="Correo" type="email" class="form-control">
                <span asp-validation-for="Correo" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-primary">Guardar</button>
            <a asp-action="Listar" asp-controller="Mantenedor" class="btn btn-warning">Volver a la lista</a>
        </form>
    </div>
</div>
```
Crear la vista para Editar
```
@model ContactoModel
@{
    ViewData["Title"] = "Editar";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="card">
    <div class="card-header">
        Editar Contacto
    </div>
    <div class="card-body">
        <form asp-action="Editar" asp-controller="Mantenedor" method="post">
            <input asp-for="IdContacto" type="hidden" class="form-control">

            <div class="mb-3">
                <label class="form-label">Nombre</label>
                <input asp-for="Nombre" type="text" class="form-control">
                <span asp-validation-for="Nombre" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label class="form-label">Telefono</label>
                <input asp-for="Telefono" type="text" class="form-control">
                <span asp-validation-for="Telefono" class="text-danger"></span>
            </div>
            <div class="mb-3">
                <label class="form-label">Correo</label>
                <input asp-for="Correo" type="email" class="form-control">
                <span asp-validation-for="Correo" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-primary">Guardar</button>
            <a asp-action="Listar" asp-controller="Mantenedor" class="btn btn-warning">Volver a la lista</a>
        </form>
    </div>
</div>
```
Crear la vista para Listar
```
@model List<ContactoModel>
@{
    ViewData["Title"] = "Listar";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<h1>Listar</h1>

<div class="card">
    <div class="card-header">
        Listar Contactos
    </div>
    <div class="card-body">
        <a asp-action="Guardar" asp-controller="Mantenedor" class="btn btn-success">Crear nuevo</a>
        <hr />
        <table class="table table-bordered">
            <thead>
                <tr>
                    <th>Nombre</th>
                    <th>Telefono</th>
                    <th>Correo</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
                @foreach (var item in Model)
                {
                    <tr>
                        <td>@item.Nombre</td>
                        <td>@item.Telefono</td>
                        <td>@item.Correo</td>
                        <td>
                            <a asp-action="Editar" asp-controller="Mantenedor" class="btn btn-primary btn-sm" asp-route-IdContacto="@item.IdContacto">Editar</a>
                            <a asp-action="Eliminar" asp-controller="Mantenedor" class="btn btn-danger asp-route-IdContacto="@item.IdContacto" btn-sm">Eliminar</a>

                        </td>
                    </tr>
                }
            </tbody>
        </table>
    </div>
</div>
```
Crear la vista para Eliminar
```
@model ContactoModel
@{
    ViewData["Title"] = "Eliminar";
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<div class="card">
    <div class="card-header">
        Eliminar Contacto
    </div>
    <div class="card-body">
        <form asp-action="Eliminar" asp-controller="Mantenedor" method="post">
            <input asp-for="IdContacto" type="hidden" class="form-control">

            <div class="alert alert-danger" role="alert">
                Deseas eliminar el contacto: @Html.DisplayTextFor(m => m.Nombre) ?
            </div>

            <button type="submit" class="btn btn-danger">Eliminar</button>
            <a asp-action="Listar" asp-controller="Mantenedor" class="btn btn-warning">Volver a la lista</a>
        </form>
    </div>
</div>
```
