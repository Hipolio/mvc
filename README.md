# mvc

MODELO

Es la capa donde se trabaja con los datos, por tanto contendrá mecanismos para acceder a la información y también para actualizar su estado. Los datos los tendremos habitualmente en una base de datos, por lo que en los modelos tendremos todas las funciones que accederán a las tablas y harán los correspondientes selects, updates, inserts, etc.

No obstante, cabe mencionar que cuando se trabaja con MCV lo habitual también es utilizar otras librerías como PDO o algún ORM como Doctrine, que nos permiten trabajar con abstracción de bases de datos y persistencia en objetos. Por ello, en vez de usar directamente sentencias SQL, que suelen depender del motor de base de datos con el que se esté trabajando, se utiliza un dialecto de acceso a datos basado en clases y objetos.

Ejemplo:

<?php 

	require '../bd/conexion_bd.php';

	class Productos extends BD_PDO
	{
		function insertar($nombre, $cant, $precio)
		{
			$this->Ejecutar_Instruccion("insert into productos(nombre,cant,precio) values('$nombre','$cant','$precio')");
			echo '<script>alert("Ha Insertado Correctamente");</script>';
		}
		function buscar($datoBuscar)
		{
			$result = $this->Ejecutar_Instruccion("select * from productos where nombre like '%$datoBuscar%'");
			return $result;
		}
		function eliminar($id)
		{
			$this->Ejecutar_Instruccion("delete from productos where idProd = '$id'");
			echo '<script>alert("Ha Eliminado Correctamente");</script>';
		}
		function modificar($nombre, $cant, $precio, $id)
		{
			$this->Ejecutar_Instruccion("update productos set nombre= '$nombre', cant= '$cant', precio= '$precio' where idProd = '$id'");
			echo '<script>alert("Ha Modificado Correctamente");</script>';
		}
		function cat_mod($id)
        {
            $cat_mod = $this->Ejecutar_Instruccion("Select * from productos where idProd = '$id'");
            return $cat_mod;
        }

		function tablaBuscar($result)
		{
			$tabla = "";

			foreach (@$result as $renglon)
			{
	         
	        $tabla.='<tr>';
	        $tabla.='<td style="border-color: white;border: 2px solid;">'.$renglon[0].'</td>';
	        $tabla.='<td style="border-color: white;border: 2px solid;">'.$renglon[1].'</td>';
	        $tabla.='<td style="border-color: white;border: 2px solid;">'.$renglon[2].'</td>';
	        $tabla.='<td style="border-color: white;border: 2px solid;">'.$renglon[3].'</td>';
	        $tabla.='<td style="border-color: white;border: 2px solid; background-color: red;"><input type="button" id="btnEliminar" name="btnEliminar" value="Eliminar" onclick="javascript: eliminar('.$renglon[0].');"></td>';
	        $tabla.='<td style="border-color: white;border: 2px solid; background-color: yellow;"><input type="button" id="btnModificar" name="btnModificar" value="Modificar" onclick="javascript: modificar('.$renglon[0].');"></td></tr>';

	        }
	        
	        return $tabla;
		}
	}

 ?>
 
 VISTA
 
 Las vistas, como su nombre nos hace entender, contienen el código de nuestra aplicación que va a producir la visualización de las interfaces de usuario, o sea, el código que nos permitirá renderizar los estados de nuestra aplicación en HTML. En las vistas nada más tenemos los códigos HTML y PHP que nos permite mostrar la salida.

En la vista generalmente trabajamos con los datos, sin embargo, no se realiza un acceso directo a éstos. Las vistas requerirán los datos a los modelos y ellas se generará la salida, tal como nuestra aplicación requiera.

Ejemplo:

<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Vista Admin</title>
	<link rel="stylesheet" href="../css/styles.css">
	<script>
        function logout()
        {
            if (confirm("¿Quieres cerrar sesión?")) 
            {
                window.location = "../logout.php"
            }
        }
	</script>

	<script type="text/javascript" src="../js/validaciones.js"></script>

</head>
<body>

    <div class="row">
    	<a href="#" onclick="javascript: logout();" style="color: white;">Cerrar Sesión</a>
    </div>

	<!-- INICIO BUSCAR EN TABLA PRODUCTOS -->
	<form id="frmBuscar" name="frmBuscar" action="../c/controladorCliente.php" method="post">
		<div class="row">
			<span>
		    <input class="balloon" id="txtNombre" name="txtNombre" type="text" placeholder="Buscar" onkeypress="return Vbuscar();" /><label for="Buscar">Buscar</label>
		  </span>
		    <input type="submit" id="btnBuscar" name="btnBuscar" class="btn btn-secondary" value="Buscar" style="margin-left: 20px;margin-top: 10px;width: 96px;">
		</div>
    </form>

    <div class="row">
	    <table style="background-color: black; border-color: white;border: 2px solid;" align="center">
	        <tr>
	            <td style="border-color: white;border: 2px solid;">Id del Producto</td>
	            <td style="border-color: white;border: 2px solid;">Nombre</td>
	            <td style="border-color: white;border: 2px solid;">Cantidad</td>
	            <td style="border-color: white;border: 2px solid;">Precio</td>
	            <td style="border-color: white;border: 2px solid;">Eliminar</td>
	            <td style="border-color: white;border: 2px solid;">Modificar</td>
	        </tr>
	        <?php echo $tabla_result; ?>
	    </table>
    </div>
</body>
</html>

CONTROLADOR

Contiene el código necesario para responder a las acciones que se solicitan en la aplicación, como visualizar un elemento, realizar una compra, una búsqueda de información, etc.

En realidad es una capa que sirve de enlace entre las vistas y los modelos, respondiendo a los mecanismos que puedan requerirse para implementar las necesidades de nuestra aplicación. Sin embargo, su responsabilidad no es manipular directamente datos, ni mostrar ningún tipo de salida, sino servir de enlace entre los modelos y las vistas para implementar las diversas necesidades del desarrollo.

EJemplo:

<?php 
    
    @$datoBuscar = $_POST['txtNombre'];
    @$nombre = $_POST['nombre'];
    @$cant = $_POST['cant'];
    @$precio = $_POST['precio'];

    require '../m/modelo.php';

    $obj_prod = new Productos();

    if (isset($_POST['btnEnviar'])) {
        if ($_POST['btnEnviar']=="Registrar") {
            $obj_prod->insertar($nombre, $cant, $precio);
        }
        else if ($_POST['btnEnviar']=="Modificar"){
            $id = @$_POST['idProd'];
            $obj_prod->modificar($nombre, $cant, $precio, $id);
        }
    }
    else if (isset($_GET['id_eliminar'])) {
        $id = $_GET['id_eliminar'];
        $obj_prod->eliminar($id);
    }
    else if (isset($_GET['id_mod'])) {
        $id = $_GET['id_mod'];
        @$cat_mod = $obj_prod->cat_mod($id);
    }


    $result = $obj_prod->buscar($datoBuscar);

    $tabla_result = $obj_prod->tablaBuscar($result);

    require '../v/vista.php';

?>
