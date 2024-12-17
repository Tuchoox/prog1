//AccesoDatos
public class AccesoDatos
{

    private string CadenaConexion = "Data Source=172.16.10.196;Initial Catalog=DGR;User ID=alumno1w1;Password=alumno1w1"; // Properties.Resources.CadenaConexionLocal;
    private SqlConnection conexion;
    private SqlCommand comando;
    public AccesoDatos()
    {
        conexion = new SqlConnection(CadenaConexion);
    }
    private void Conectar()
    {
        conexion.Open();
        comando = new SqlCommand();
        comando.Connection = conexion;
        comando.CommandType = CommandType.Text;
    }
    public void Desconectar()
    {
        conexion.Close();
    }
    public DataTable ConsultarTabla(string nombreTabla)
    {
        DataTable tabla = new DataTable();
        this.Conectar();
        comando.CommandText = "SELECT * FROM " + nombreTabla;
        tabla.Load(comando.ExecuteReader());
        this.Desconectar();
        return tabla;
    }
    public DataTable ConsultarTabla2(string nombreTabla)
    {
        DataTable tabla = new DataTable();
        this.Conectar();
        comando.CommandText = "SELECT " + nombreTabla;
        tabla.Load(comando.ExecuteReader());
        this.Desconectar();
        return tabla;
    }
    public DataTable ConsultarBD(string consultaSQL)
    {
        DataTable tabla = new DataTable();
        this.Conectar();
        comando.CommandText = consultaSQL;
        tabla.Load(comando.ExecuteReader());
        this.Desconectar();
        return tabla;
    }
    public int ActualizarBD(string consultaSQL)
    {
        int filasAfectadas = 0;
        this.Conectar();
        comando.CommandText = consultaSQL;
        filasAfectadas = comando.ExecuteNonQuery();
        this.Desconectar();
        return filasAfectadas;
    }
    public int ActualizarBD(string consultaSQL, List<Parametro> lista)
    {
        int filasAfectadas = 0;
        this.Conectar();
        comando.CommandText = consultaSQL;
        foreach (Parametro p in lista)
        {
            comando.Parameters.AddWithValue(p.Nombre, p.Valor);
        }
        filasAfectadas = comando.ExecuteNonQuery();
        this.Desconectar();
        return filasAfectadas;
    }

}



-
-
-
-
-
-
-



//Parametro
public class Parametro
{

    private string nombre;
    private object valor;
    public string Nombre
    {
        get { return nombre; }
        set { nombre = value; }
    }
    public object Valor
    {
        get { return valor; }
        set { valor = value; }
    }
    public Parametro(string nombre, object valor)
    {
        this.nombre = nombre;
        this.valor = valor;
    }

}




-
-
-
-
-
-




//categoriaDao
public class CategoriaDao
{
    private AccesoDatos _acceso = new AccesoDatos();

    public DataTable CargarCategorias()
    {
        DataTable dt = _acceso.ConsultarTabla("Categorias");
        return dt;
    }
}




-
-
-
-
-
-



//InmuebleDao
public class InmuebleDao
{
    private AccesoDatos _acceso = new AccesoDatos();

    public List<Inmueble> CargarInmuebles()
    {
        DataTable dt = _acceso.ConsultarTabla2(" id_inmueble, titular, nomenclatura, id_categoria, n_categoria, valuacion from Inmuebles i join categorias c on i.categoria = c.id_categoria ");
        List<Inmueble> inmuebles = new List<Inmueble>();
        foreach (DataRow dr in dt.Rows)
        {
            Categoria cat = new Categoria();
            cat.IdCategoria = Convert.ToInt32(dr["id_categoria"]);
            cat.nCategoria = dr["n_categoria"].ToString();

            Inmueble inmueble = new Inmueble();
            inmueble.IdInmueble = Convert.ToInt32(dr["id_inmueble"]);
            inmueble.Titular = dr["titular"].ToString();
            inmueble.Nomenclatura = Convert.ToInt32(dr["nomenclatura"]);
            inmueble.Categoria = cat; //aca tengo que mostrar rural
            inmueble.Valuacion = Convert.ToDouble(dr["valuacion"]);
            inmuebles.Add(inmueble);
        }

        return inmuebles;
    }

    internal void ActualizarInmu(Inmueble inmueble)
    {
        string consulta = "update Inmuebles set titular = '" + inmueble.Titular + "', nomenclatura = " + inmueble.Nomenclatura + ", tipo = " + inmueble.Tipo+ ", categoria = " + inmueble.Categoria.IdCategoria + ", sup_terreno= " + inmueble.SupTerreno + ", sup_edificada= " + inmueble.SupEdificada + ", valuacion= " + inmueble.Valuacion + " where id_inmueble = " + inmueble.IdInmueble;
        int filasAfectadas = _acceso.ActualizarBD(consulta);
        if (filasAfectadas >0)
        {
            MessageBox.Show("Inmueble Actualizado");
        } else
        {
            MessageBox.Show("Error al actualizar el inmueble");
        }
    }

    internal Inmueble CargarInmueble(int codigo)
    {
        string query = $" id_inmueble, titular, nomenclatura, tipo, id_categoria, n_categoria, valuacion, sup_terreno, sup_edificada from Inmuebles i join categorias c on i.categoria = c.id_categoria where id_inmueble = {codigo}";
        DataTable dt = _acceso.ConsultarTabla2(query);
        if (dt.Rows.Count>0)
        {
            Inmueble inmueble = new Inmueble
            {
                IdInmueble = codigo,
                Titular = dt.Rows[0]["titular"].ToString(),
                Nomenclatura = Convert.ToInt32(dt.Rows[0]["nomenclatura"]),
                Tipo = Convert.ToInt32(dt.Rows[0]["tipo"]),
                Categoria = new Categoria
                {
                    IdCategoria = Convert.ToInt32(dt.Rows[0]["id_categoria"]),
                    nCategoria = dt.Rows[0]["n_categoria"].ToString()
                },
                Valuacion = Convert.ToDouble(dt.Rows[0]["valuacion"]),
                SupTerreno = Convert.ToDouble(dt.Rows[0]["sup_terreno"]),
                SupEdificada = Convert.ToDouble(dt.Rows[0]["sup_edificada"])
            };
            return inmueble;
        }
        return null;
    }
}




-
-
-
-
-
-



//Servicio
public class Servicio
{
    private CategoriaDao _categoriaDao = new CategoriaDao();
    private InmuebleDao _inmuebleDao = new InmuebleDao();

    public DataTable CargarCategorias()
    {
        return _categoriaDao.CargarCategorias();
    }

    public List<Inmueble> CargarInmuebles()
    {
        return _inmuebleDao.CargarInmuebles();
    }

    internal void ActualizarInmueble(Inmueble inmueble)
    {
        _inmuebleDao.ActualizarInmu(inmueble);
    }

    internal Inmueble CargarInmueble(int codigo)
    {
        return _inmuebleDao.CargarInmueble(codigo);
    }
}


-
-
-
-
-
-


//FrmInmuebles
public partial class FrmInmuebles : Form
{

    private Servicio _servicio = new Servicio();
    private int codigo;
    public FrmInmuebles()
    {
        InitializeComponent();
    }

    private void FrmInmuebles_Load(object sender, EventArgs e)
    {
        CargarCombo();
        CargarInmuebles();
    }

    private void CargarInmuebles()
    {
       List<Inmueble> inmuebles = _servicio.CargarInmuebles();
        lstInmuebles.DataSource = inmuebles;

    }

    private void CargarCombo()
    {
        DataTable dt = _servicio.CargarCategorias();
        cboCategoria.DataSource = dt;
        cboCategoria.ValueMember = dt.Columns[0].ColumnName;
        cboCategoria.DisplayMember = dt.Columns[1].ColumnName;
        cboCategoria.SelectedIndex = -1;
    }

    private void btnNuevo_Click(object sender, EventArgs e)
    {
       
            if (lstInmuebles.SelectedItem is Inmueble selectedInmueble)
            {
                codigo = selectedInmueble.IdInmueble;
            }
            else
            {
                MessageBox.Show("Por favor, seleccione un inmueble de la lista.");
            }

        Inmueble inmuebleEditar = CargarInmueble();
        txtTitular.Text = inmuebleEditar.Titular;
        txtNomenclatura.Text = inmuebleEditar.Nomenclatura.ToString();
        rbtRural.Checked = inmuebleEditar.Tipo == 1;
        rbtUrbano.Checked = inmuebleEditar.Tipo == 2;
        cboCategoria.SelectedValue = inmuebleEditar.Categoria.IdCategoria;
        txtTerreno.Text = inmuebleEditar.SupTerreno.ToString();
        txtEdificada.Text = inmuebleEditar.SupEdificada.ToString();
        txtValuacion.Text = inmuebleEditar.Valuacion.ToString();
        btnGrabar.Enabled = true;


    }
    public bool Validar()
    {
        if (string.IsNullOrEmpty(txtTitular.Text))
        {
            MessageBox.Show("Debe ingresar el titular del inmueble.");
            txtTitular.Focus();
            return false;
        }
        if (!rbtRural.Checked && !rbtUrbano.Checked)
        {
            MessageBox.Show("Debe seleccionar el tipo de inmueble.");
            rbtRural.Focus();
            return false;
        }
        if (string.IsNullOrEmpty(txtNomenclatura.Text) || !int.TryParse(txtNomenclatura.Text, out _))
        {
            MessageBox.Show("Debe ingresar una nomenclatura válida (número entero).");
            txtNomenclatura.Focus();
            return false;
        }
        if (string.IsNullOrEmpty(txtTerreno.Text) || !double.TryParse(txtTerreno.Text, out _))
        {
            MessageBox.Show("Debe ingresar una superficie del terreno válida (número decimal).");
            txtTerreno.Focus();
            return false;
        }
        if (string.IsNullOrEmpty(txtEdificada.Text) || !double.TryParse(txtEdificada.Text, out _))
        {
            MessageBox.Show("Debe ingresar una superficie edificada válida (número decimal).");
            txtEdificada.Focus();
            return false;
        }
        if (string.IsNullOrEmpty(txtValuacion.Text) || !double.TryParse(txtValuacion.Text, out _))
        {
            MessageBox.Show("Debe ingresar una valuación válida (número decimal).");
            txtValuacion.Focus();
            return false;
        }
        if (cboCategoria.SelectedIndex == -1)
        {
            MessageBox.Show("Debe seleccionar una categoría.");
            cboCategoria.Focus();
            return false;
        }
        if (cboCategoria.SelectedIndex == 1)
        {
            txtEdificada.Text = "";
            txtEdificada.Enabled = false;
        }
        return true;
    }


    private Inmueble CargarInmueble()
    {
        Inmueble inmueble = _servicio.CargarInmueble(codigo);
        return inmueble;
    }


    private void btnGrabar_Click(object sender, EventArgs e)
    {
        if (Validar())
        {
            Inmueble inmueble = CrearInmueble();
            _servicio.ActualizarInmueble(inmueble);
            CargarInmuebles();
            Limpiar();
        }
        
    }

    private void Limpiar()
    {
        btnGrabar.Enabled = false;
        txtEdificada.Text = "";
        txtNomenclatura.Text = string.Empty;
        txtTerreno.Text = string.Empty;
        txtTitular.Text = string.Empty;
        rbtRural.Checked = false;
        rbtUrbano.Checked = false;
        cboCategoria.SelectedIndex = -1;
    txtValuacion.Text = string.Empty;
    }

    private Inmueble CrearInmueble()
    {
        int tipo = 0;
        if (rbtRural.Checked)
        {
            tipo = 2;
        }
        else if (rbtUrbano.Checked)
        {
            tipo = 1;
        }

        Inmueble inmu = new Inmueble
        {
            IdInmueble = codigo,
            Titular = txtTitular.Text,
            Nomenclatura = Convert.ToInt32(txtNomenclatura.Text),
            Tipo = tipo,
            Categoria = new Categoria
            {
                IdCategoria = Convert.ToInt32(cboCategoria.SelectedValue),
                nCategoria = cboCategoria.Text
            },
            SupTerreno = Convert.ToDouble(txtTerreno.Text),
            SupEdificada = Convert.ToDouble(txtEdificada.Text),
            Valuacion = Convert.ToDouble(txtValuacion.Text)
        };
        return inmu;
    }

    private void btnSalir_Click(object sender, EventArgs e)
    {
        if (MessageBox.Show("¿Esta seguro de que desea salir?", "SALIR", MessageBoxButtons.YesNo, MessageBoxIcon.Warning, MessageBoxDefaultButton.Button2) == DialogResult.Yes)
        {
            this.Close();
        }
            
    }
    private void txtNomenclatura_KeyPress(object sender, KeyPressEventArgs e)
    {
    }

    private void txtValuacion_TextChanged(object sender, EventArgs e)
    {
    }

    private void txtNomenclatura_TextChanged(object sender, EventArgs e)
    {

    }
}



-
-
-
-
-



//Inmueble
public class Inmueble
 {
 
     public int IdInmueble { get; set; }
     public string Titular { get; set; }
     public int Nomenclatura { get; set; }
     public int Tipo { get; set; }
     public Categoria Categoria { get; set; }
     public double SupTerreno { get; set; }
     public double SupEdificada { get; set; }
     public double Valuacion { get; set; }

     public Inmueble()
     {
         IdInmueble = 0;
         Titular = string.Empty;
         Nomenclatura = 0;
         Tipo = 0;
         Categoria = new Categoria();
         SupTerreno = 0;
         SupEdificada = 0;
         Valuacion = 0;
     }

     public Inmueble(int idInmueble, string titular, int nomenclatura, int tipo, Categoria categoria, double supTerreno, double supEdificada, double valuacion)
     {
         IdInmueble = idInmueble;
         Titular = titular;
         Nomenclatura = nomenclatura;
         Tipo = tipo;
         Categoria = categoria;
         SupTerreno = supTerreno;
         SupEdificada = supEdificada;
         Valuacion = valuacion;
     }

     public override string ToString()
     {
         return Nomenclatura + " - " + Titular + " - " +Categoria.nCategoria +" - "+Valuacion;
     }
 }
