<!DOCTYPE html>
<html lang="es">
<head>
<script src="https://www.gstatic.com/firebasejs/9.6.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.0/firebase-firestore-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.6.0/firebase-auth-compat.js"></script>
  <meta charset="UTF-8">
  <title>Gestión de Almacén - Completo</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.17.1/xlsx.full.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4; color: #333; margin: 0; padding: 0;
    }
    .login-container {
      width: 300px; margin: 100px auto; padding: 20px;
      background: #fff; border-radius: 5px; border: 1px solid #ccc; text-align: center;
    }
    #error-message { color: red; display: none; margin-top: 5px; }

    #tabs { text-align: center; margin: 20px; }
    .tab-button {
      padding: 10px 15px; margin: 5px; background: #ddd; border: 1px solid #ccc;
      border-radius: 5px; cursor: pointer;
    }
    .tab-button.active { background: #4CAF50; color: #fff; }

    .form-container {
      width: 1000px; margin: 20px auto; padding: 15px;
      border: 1px solid #ccc; border-radius: 5px; background: #fff;
    }
    .form-container h2 { text-align: center; }
    input, select, button {
      display: block; width: 100%; margin: 10px 0; padding: 10px;
      border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box;
    }
    button { background: #4CAF50; color: #fff; border: none; cursor: pointer; }
    button:hover { background: #45a049; }
    button.secondary { background: #3498db; }
    button.secondary:hover { background: #2980b9; }
    button.danger { background: #e74c3c; }
    button.danger:hover { background: #c0392b; }

    .tab-content { display: none; }
    .tab-content.active { display: block; }

    #productosDisponibles div {
      border: 1px solid #ccc; background: #fff; padding: 10px; margin: 5px 0;
    }
    #historialMovimientos {
      max-height: 250px; overflow-y: auto; border: 1px solid #ccc; background: #fff; padding: 10px;
    }
    #consultaResult { margin-top: 10px; }

    /* Estilos para imágenes */
    .imagen-container {
      margin: 10px 0;
      text-align: center;
    }
    .imagen-producto {
      max-width: 200px;
      max-height: 150px;
      border: 1px solid #ddd;
      border-radius: 4px;
      object-fit: contain;
    }
    .input-imagen {
      border: 2px dashed #ccc;
      padding: 10px;
      text-align: center;
      cursor: pointer;
      margin: 10px 0;
    }
    .preview-imagen {
      position: relative;
      display: inline-block;
    }
    .eliminar-imagen {
      position: absolute;
      top: -10px;
      right: -10px;
      background: #e74c3c;
      color: white;
      border-radius: 50%;
      width: 20px;
      height: 20px;
      line-height: 20px;
      text-align: center;
      cursor: pointer;
      font-weight: bold;
    }

    /* Estilos para tablas */
    table {
      width: 100%;
      border-collapse: collapse;
      margin: 10px 0;
    }
    th, td {
      padding: 8px;
      text-align: left;
      border-bottom: 1px solid #ddd;
      vertical-align: middle;
    }
    th {
      background-color: #f2f2f2;
    }
    tr:hover {
      background-color: #f5f5f5;
    }

    /* Paginación */
    .pagination {
      display: flex;
      justify-content: center;
      margin: 20px 0;
    }
    .pagination button {
      width: auto;
      margin: 0 5px;
      padding: 5px 10px;
    }

    /* Filtros de búsqueda */
    .search-filters {
      display: flex;
      gap: 10px;
      margin-bottom: 15px;
    }
    .search-filters input, .search-filters select {
      flex: 1;
      margin: 0;
    }

    /* Notificaciones */
    .notification {
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 15px;
      background: #2ecc71;
      color: white;
      border-radius: 5px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
      z-index: 1000;
      display: none;
    }

    /* Estilos para gráficos */
    .charts-container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
      gap: 20px;
      margin-top: 20px;
    }

    .chart-box {
      background: white;
      padding: 15px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      position: relative;
    }

    .chart-box canvas {
      max-height: 400px;
      width: 100% !important;
    }

    .chart-box button {
      margin-top: 10px;
      width: auto;
      padding: 8px 15px;
    }
  </style>
</head>
<body>

  <!-- Notificación -->
  <div class="notification" id="notification"></div>

  <!-- Login -->
  <div class="login-container" id="loginScreen">
    <h3>Iniciar sesión</h3>
    <form id="loginForm">
      <label>Usuario:</label>
      <input type="text" id="usuario" required>

      <label>Contraseña:</label>
      <input type="password" id="password" required>

      <button type="submit">Iniciar sesión</button>
    </form>
    <div id="error-message">Usuario o contraseña incorrectos</div>
  </div>

  <!-- Pestañas -->
  <div id="tabs" style="display:none;">
    <button class="tab-button active" data-target="registro" onclick="showTab('registro')">Registrar</button>
    <button class="tab-button" data-target="consulta" onclick="showTab('consulta')">Consultar</button>
    <button class="tab-button" data-target="almacen" onclick="showTab('almacen')">Almacén</button>
    <button class="tab-button" data-target="historial" onclick="showTab('historial')">Historial</button>
    <button class="tab-button" data-target="eliminar" onclick="showTab('eliminar')">Eliminar</button>
    <button class="tab-button" data-target="editar" onclick="showTab('editar')">Editar</button>
    <button class="tab-button" data-target="estadisticas" onclick="showTab('estadisticas')">Estadísticas</button>
  </div>

  <!-- Registrar Producto -->
  <div class="form-container tab-content active" id="registro">
    <h2>Registrar Producto</h2>
    <form id="registroProductoForm">
      <label>Fecha:</label>
      <input type="date" id="fecha" required>

      <label>Usuario:</label>
      <input type="text" id="usuarioRegistro" required>

      <label>Producto:</label>
      <input type="text" id="producto" required>

      <label>Cantidad:</label>
      <input type="number" id="cantidad" required min="1">

      <label>Estantería:</label>
      <input type="text" id="estanteria" required>

      <label>Balda:</label>
      <input type="text" id="balda" required>

      <label>Obra:</label>
      <input type="text" id="obra" required>

      <label>Imagen del Producto (JPEG/PNG, max 2MB):</label>
      <div class="input-imagen" onclick="document.getElementById('fotoProducto').click()">
        <div id="previewContainer">Haz clic para subir imagen</div>
        <input type="file" id="fotoProducto" accept="image/jpeg, image/png" style="display:none;" onchange="previsualizarImagen(event)">
      </div>

      <button type="submit">Registrar</button>
    </form>
  </div>

  <!-- Consultar Producto -->
  <div class="form-container tab-content" id="consulta">
    <h2>Consultar Producto</h2>
    <div class="search-filters">
      <input type="text" id="productoConsulta" placeholder="Nombre del producto">
      <input type="text" id="ubicacionConsulta" placeholder="Estantería/Balda">
      <select id="obraConsulta">
        <option value="">Todas las obras</option>
      </select>
    </div>
    <button onclick="consultarProducto()">Consultar</button>
    <div id="consultaResult"></div>
    <div class="pagination" id="paginationConsulta" style="display:none;">
      <button onclick="changePage(-1)" id="prevPage">Anterior</button>
      <span id="pageInfo">Página 1</span>
      <button onclick="changePage(1)" id="nextPage">Siguiente</button>
    </div>
  </div>

  <!-- Almacén -->
  <div class="form-container tab-content" id="almacen">
    <h2>Productos en Almacén</h2>
    <div class="search-filters">
      <input type="text" id="filtroAlmacen" placeholder="Filtrar productos...">
      <select id="ordenAlmacen">
        <option value="nombre">Ordenar por nombre</option>
        <option value="cantidad">Ordenar por cantidad</option>
        <option value="ubicacion">Ordenar por ubicación</option>
      </select>
    </div>
    <div id="productosDisponibles"></div>
    <div class="pagination" id="paginationAlmacen" style="display:none;">
      <button onclick="changeAlmacenPage(-1)" id="prevAlmacenPage">Anterior</button>
      <span id="almacenPageInfo">Página 1</span>
      <button onclick="changeAlmacenPage(1)" id="nextAlmacenPage">Siguiente</button>
    </div>
  </div>

  <!-- Historial -->
  <div class="form-container tab-content" id="historial">
    <h2>Historial de Movimientos</h2>
    <div class="search-filters">
      <input type="text" id="filtroHistorial" placeholder="Filtrar historial...">
      <input type="date" id="fechaDesdeHistorial">
      <input type="date" id="fechaHastaHistorial">
    </div>
    <div id="historialMovimientos"></div>
    <div class="pagination" id="paginationHistorial" style="display:none;">
      <button onclick="changeHistorialPage(-1)" id="prevHistorialPage">Anterior</button>
      <span id="historialPageInfo">Página 1</span>
      <button onclick="changeHistorialPage(1)" id="nextHistorialPage">Siguiente</button>
    </div>
    <button onclick="generarAlbaran()">Generar Albarán</button>
    <button class="secondary" onclick="enviarAlbaran()">Enviar Albarán</button>
    <br><br>
    <button class="secondary" onclick="hacerBackup()">Hacer Copia</button>
    <input type="file" accept=".json" onchange="cargarBackup(event)">
  </div>

  <!-- Eliminar -->
  <div class="form-container tab-content" id="eliminar">
    <h2>Eliminar Producto</h2>
    <label>Producto:</label>
    <select id="productoEliminar"></select>
    <button class="danger" onclick="eliminarProducto()">Eliminar</button>
  </div>

  <!-- Editar -->
  <div class="form-container tab-content" id="editar">
    <h2>Editar Producto</h2>
    <label>Producto:</label>
    <select id="productoEditar" onchange="cargarDetallesEdicion(this.value)"></select>

    <label>Cantidad:</label>
    <input type="number" id="cantidadEditar" min="1">

    <label>Estantería:</label>
    <input type="text" id="estanteriaEditar">

    <label>Balda:</label>
    <input type="text" id="baldaEditar">

    <label>Obra:</label>
    <input type="text" id="obraEditar">

    <label>Imagen del Producto (JPEG/PNG, max 2MB):</label>
    <div class="input-imagen" onclick="document.getElementById('fotoEditar').click()">
      <div id="previewEditar">Haz clic para cambiar imagen</div>
      <input type="file" id="fotoEditar" accept="image/jpeg, image/png" style="display:none;" onchange="previsualizarImagenEdicion(event)">
    </div>

    <button onclick="editarProducto()">Guardar</button>
    <button class="danger" onclick="eliminarImagenProducto()">Eliminar Imagen</button>
  </div>

  <!-- Estadísticas -->
  <div class="form-container tab-content" id="estadisticas">
    <h2>Estadísticas y Gráficos</h2>
    
    <!-- Selector de Tipo de Gráfico -->
    <div class="search-filters">
      <select id="tipoGrafico" onchange="actualizarGraficos()">
        <option value="stock">Stock por Producto</option>
        <option value="ubicacion">Distribución por Ubicación</option>
        <option value="movimientos">Movimientos Diarios</option>
      </select>
      <button onclick="exportarGraficos()" class="secondary">Exportar Reporte</button>
    </div>

    <!-- Contenedores de Gráficos -->
    <div class="charts-container">
      <div class="chart-box">
        <canvas id="mainChart"></canvas>
        <button onclick="exportarChart('mainChart')" class="secondary">Exportar Gráfico</button>
      </div>
      <div class="chart-box">
        <canvas id="secondaryChart"></canvas>
        <button onclick="exportarChart('secondaryChart')" class="secondary">Exportar Gráfico</button>
      </div>
    </div>
  </div>

  <script>
    // =============================
    // Variables globales
    // =============================
    let productos = JSON.parse(localStorage.getItem("productos")) || [];
    let historial = JSON.parse(localStorage.getItem("historialMovimientos")) || [];
    let fotoTemporal = null;
    let fotoEdicionTemporal = null;
    let currentPage = 1;
    let currentAlmacenPage = 1;
    let currentHistorialPage = 1;
    const itemsPerPage = 10;
    let filteredProducts = [];
    let filteredHistorial = [];
    let mainChartInstance = null;
    let secondaryChartInstance = null;

    // =============================
    // Funciones para manejo de imágenes
    // =============================
    function previsualizarImagen(event) {
      const file = event.target.files[0];
      if (!file) return;
      
      // Validar tipo de archivo
      const validTypes = ['image/jpeg', 'image/png'];
      if (!validTypes.includes(file.type)) {
        showNotification('Error: Solo se permiten imágenes JPEG o PNG', 'error');
        event.target.value = "";
        return;
      }

      // Validar tamaño
      if (file.size > 2 * 1024 * 1024) {
        showNotification('Error: La imagen debe ser menor a 2MB', 'error');
        event.target.value = "";
        return;
      }

      // Comprimir imagen antes de mostrar
      compressImage(file, 200, 200, 0.7)
        .then(compressedFile => {
          const reader = new FileReader();
          reader.onload = function(e) {
            fotoTemporal = e.target.result;
            document.getElementById('previewContainer').innerHTML = `
              <div class="preview-imagen">
                <img src="${e.target.result}" class="imagen-producto">
                <div class="eliminar-imagen" onclick="eliminarImagen()">×</div>
              </div>
            `;
          };
          reader.readAsDataURL(compressedFile);
        })
        .catch(error => {
          console.error("Error al comprimir imagen:", error);
          showNotification('Error al procesar la imagen', 'error');
          event.target.value = "";
        });
    }

    function previsualizarImagenEdicion(event) {
      const file = event.target.files[0];
      if (!file) return;
      
      // Validar tipo de archivo
      const validTypes = ['image/jpeg', 'image/png'];
      if (!validTypes.includes(file.type)) {
        showNotification('Error: Solo se permiten imágenes JPEG o PNG', 'error');
        event.target.value = "";
        return;
      }

      // Validar tamaño
      if (file.size > 2 * 1024 * 1024) {
        showNotification('Error: La imagen debe ser menor a 2MB', 'error');
        event.target.value = "";
        return;
      }

      // Comprimir imagen antes de mostrar
      compressImage(file, 200, 200, 0.7)
        .then(compressedFile => {
          const reader = new FileReader();
          reader.onload = function(e) {
            fotoEdicionTemporal = e.target.result;
            document.getElementById('previewEditar').innerHTML = `
              <div class="preview-imagen">
                <img src="${e.target.result}" class="imagen-producto">
                <div class="eliminar-imagen" onclick="eliminarImagenEdicion()">×</div>
              </div>
            `;
          };
          reader.readAsDataURL(compressedFile);
        })
        .catch(error => {
          console.error("Error al comprimir imagen:", error);
          showNotification('Error al procesar la imagen', 'error');
          event.target.value = "";
        });
    }

    function compressImage(file, maxWidth, maxHeight, quality) {
      return new Promise((resolve, reject) => {
        const img = new Image();
        const reader = new FileReader();
        
        reader.onload = function(e) {
          img.src = e.target.result;
        };
        
        img.onload = function() {
          const canvas = document.createElement('canvas');
          let width = img.width;
          let height = img.height;
          
          // Calcular nuevas dimensiones manteniendo aspect ratio
          if (width > height) {
            if (width > maxWidth) {
              height *= maxWidth / width;
              width = maxWidth;
            }
          } else {
            if (height > maxHeight) {
              width *= maxHeight / height;
              height = maxHeight;
            }
          }
          
          canvas.width = width;
          canvas.height = height;
          
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, width, height);
          
          canvas.toBlob(blob => {
            if (!blob) {
              reject(new Error('Error al comprimir imagen'));
              return;
            }
            resolve(blob);
          }, file.type, quality);
        };
        
        img.onerror = function() {
          reject(new Error('Error al cargar imagen'));
        };
        
        reader.readAsDataURL(file);
      });
    }

    function eliminarImagen() {
      fotoTemporal = null;
      document.getElementById('fotoProducto').value = '';
      document.getElementById('previewContainer').innerHTML = 'Haz clic para subir imagen';
    }

    function eliminarImagenEdicion() {
      fotoEdicionTemporal = null;
      document.getElementById('fotoEditar').value = '';
      document.getElementById('previewEditar').innerHTML = 'Haz clic para cambiar imagen';
    }

    function eliminarImagenProducto() {
      const producto = document.getElementById("productoEditar").value;
      const p = productos.find(x => x.producto === producto);
      if (p && p.imagen) {
        p.imagen = null;
        fotoEdicionTemporal = null;
        document.getElementById('previewEditar').innerHTML = 'Haz clic para cambiar imagen';
        document.getElementById('fotoEditar').value = '';
        guardarDatos();
        showNotification('Imagen eliminada correctamente');
      } else {
        showNotification('El producto no tiene imagen', 'error');
      }
    }

    // =============================
    // Funciones de UI/UX
    // =============================
    function showNotification(message, type = 'success') {
      const notification = document.getElementById('notification');
      notification.textContent = message;
      notification.style.display = 'block';
      notification.style.backgroundColor = type === 'error' ? '#e74c3c' : '#2ecc71';
      
      setTimeout(() => {
        notification.style.display = 'none';
      }, 3000);
    }

    function showTab(id) {
      const tabs = document.querySelectorAll(".tab-content");
      const btns = document.querySelectorAll(".tab-button");
      tabs.forEach(t => t.classList.remove("active"));
      btns.forEach(b => b.classList.remove("active"));

      document.getElementById(id).classList.add("active");
      const btn = [...btns].find(b => b.dataset.target === id);
      if (btn) btn.classList.add("active");

      // Actualizar la vista cuando se cambia de pestaña
      if (id === 'almacen') {
        mostrarAlmacen();
      } else if (id === 'historial') {
        mostrarHistorial();
      } else if (id === 'consulta') {
        cargarObrasConsulta();
      } else if (id === 'estadisticas') {
        actualizarGraficos();
      }
    }

    // =============================
    // Funciones principales
    // =============================
    document.getElementById("registroProductoForm").addEventListener("submit", (e) => {
      e.preventDefault();
      const fecha = document.getElementById("fecha").value;
      const usuarioReg = document.getElementById("usuarioRegistro").value;
      const prodName = document.getElementById("producto").value.trim();
      const cant = parseInt(document.getElementById("cantidad").value);
      const est = document.getElementById("estanteria").value.trim();
      const bal = document.getElementById("balda").value.trim();
      const obr = document.getElementById("obra").value.trim();

      if (cant <= 0) {
        showNotification("La cantidad debe ser mayor que cero", "error");
        return;
      }

      // Si existe, aumentar, si no, crear
      const existe = productos.find(p => p.producto.toLowerCase() === prodName.toLowerCase());
      if (existe) {
        existe.cantidad += cant;
      } else {
        productos.push({ 
          fecha, 
          usuario: usuarioReg, 
          producto: prodName, 
          cantidad: cant, 
          estanteria: est, 
          balda: bal, 
          obra: obr,
          imagen: fotoTemporal
        });
      }
      
      historial.push({ 
        tipo: "Ingreso", 
        fecha, 
        usuario: usuarioReg, 
        producto: prodName, 
        cantidad: cant, 
        estanteria: est, 
        balda: bal, 
        obra: obr
      });
      
      guardarDatos();
      showNotification("Producto registrado correctamente");
      e.target.reset();
      eliminarImagen();
      mostrarAlmacen();
      mostrarHistorial();
      cargarSelects();
      cargarObrasConsulta();
    });

    function mostrarAlmacen() {
      const div = document.getElementById("productosDisponibles");
      const filtro = document.getElementById("filtroAlmacen").value.toLowerCase();
      const orden = document.getElementById("ordenAlmacen").value;
      
      // Filtrar productos
      filteredProducts = productos.filter(p => 
        p.producto.toLowerCase().includes(filtro) ||
        p.estanteria.toLowerCase().includes(filtro) ||
        p.balda.toLowerCase().includes(filtro) ||
        p.obra.toLowerCase().includes(filtro)
      );
      
      // Ordenar productos
      switch(orden) {
        case 'cantidad':
          filteredProducts.sort((a, b) => b.cantidad - a.cantidad);
          break;
        case 'ubicacion':
          filteredProducts.sort((a, b) => {
            const locA = `${a.estanteria}${a.balda}`;
            const locB = `${b.estanteria}${b.balda}`;
            return locA.localeCompare(locB);
          });
          break;
        default: // 'nombre'
          filteredProducts.sort((a, b) => a.producto.localeCompare(b.producto));
      }
      
      // Paginación
      const totalPages = Math.ceil(filteredProducts.length / itemsPerPage);
      const pagination = document.getElementById("paginationAlmacen");
      
      if (filteredProducts.length > itemsPerPage) {
        pagination.style.display = 'flex';
        document.getElementById("almacenPageInfo").textContent = `Página ${currentAlmacenPage} de ${totalPages}`;
        document.getElementById("prevAlmacenPage").disabled = currentAlmacenPage <= 1;
        document.getElementById("nextAlmacenPage").disabled = currentAlmacenPage >= totalPages;
      } else {
        pagination.style.display = 'none';
        currentAlmacenPage = 1;
      }
      
      // Mostrar productos para la página actual
      const startIdx = (currentAlmacenPage - 1) * itemsPerPage;
      const endIdx = startIdx + itemsPerPage;
      const productsToShow = filteredProducts.slice(startIdx, endIdx);
      
      div.innerHTML = "";
      if (productsToShow.length === 0) {
        div.innerHTML = "<p>No se encontraron productos</p>";
        return;
      }
      
      productsToShow.forEach(p => {
        div.innerHTML += `
          <div>
            <strong>${p.producto}</strong> - Cant: ${p.cantidad} (Est: ${p.estanteria}, Bal: ${p.balda}, Obra: ${p.obra})
            ${p.imagen ? `<div class="imagen-container"><img src="${p.imagen}" class="imagen-producto" alt="${p.producto}"></div>` : ''}
          </div>
        `;
      });
    }

    function changeAlmacenPage(delta) {
      const totalPages = Math.ceil(filteredProducts.length / itemsPerPage);
      currentAlmacenPage += delta;
      
      if (currentAlmacenPage < 1) currentAlmacenPage = 1;
      if (currentAlmacenPage > totalPages) currentAlmacenPage = totalPages;
      
      mostrarAlmacen();
    }

    function cargarDetallesEdicion(prod) {
      const p = productos.find(x => x.producto === prod);
      if (p) {
        document.getElementById("cantidadEditar").value = p.cantidad;
        document.getElementById("estanteriaEditar").value = p.estanteria;
        document.getElementById("baldaEditar").value = p.balda;
        document.getElementById("obraEditar").value = p.obra;
        fotoEdicionTemporal = p.imagen || null;
        
        const preview = document.getElementById('previewEditar');
        preview.innerHTML = p.imagen ? 
          `<div class="preview-imagen">
            <img src="${p.imagen}" class="imagen-producto">
            <div class="eliminar-imagen" onclick="eliminarImagenEdicion()">×</div>
          </div>` : 
          'Haz clic para cambiar imagen';
      }
    }

    function editarProducto() {
      const sel = document.getElementById("productoEditar").value;
      const c = parseInt(document.getElementById("cantidadEditar").value);
      const est = document.getElementById("estanteriaEditar").value.trim();
      const bal = document.getElementById("baldaEditar").value.trim();
      const obr = document.getElementById("obraEditar").value.trim();

      const p = productos.find(x => x.producto === sel);
      if (!p) {
        showNotification("No existe el producto", "error");
        return;
      }
      
      if (c <= 0) {
        showNotification("La cantidad debe ser mayor que cero", "error");
        return;
      }
      
      p.cantidad = c;
      p.estanteria = est;
      p.balda = bal;
      p.obra = obr;
      if (fotoEdicionTemporal !== null) {
        p.imagen = fotoEdicionTemporal;
      }
      
      guardarDatos();
      showNotification(`Producto ${sel} editado`);
      mostrarAlmacen();
      mostrarHistorial();
      cargarSelects();
      cargarObrasConsulta();
    }

    function consultarProducto() {
      const nombre = document.getElementById("productoConsulta").value.toLowerCase().trim();
      const ubicacion = document.getElementById("ubicacionConsulta").value.toLowerCase().trim();
      const obra = document.getElementById("obraConsulta").value.toLowerCase().trim();
      
      const div = document.getElementById("consultaResult");
      div.innerHTML = "";
      
      const encontrados = productos.filter(p => 
        (nombre === "" || p.producto.toLowerCase().includes(nombre)) &&
        (ubicacion === "" || 
          p.estanteria.toLowerCase().includes(ubicacion) || 
          p.balda.toLowerCase().includes(ubicacion)) &&
        (obra === "" || p.obra.toLowerCase() === obra)
      );
      
      if (encontrados.length === 0) {
        div.innerHTML = "<p>No se encontraron productos</p>";
        document.getElementById("paginationConsulta").style.display = "none";
        return;
      }
      
      // Paginación
      const totalPages = Math.ceil(encontrados.length / itemsPerPage);
      const pagination = document.getElementById("paginationConsulta");
      
      if (encontrados.length > itemsPerPage) {
        pagination.style.display = 'flex';
        document.getElementById("pageInfo").textContent = `Página ${currentPage} de ${totalPages}`;
        document.getElementById("prevPage").disabled = currentPage <= 1;
        document.getElementById("nextPage").disabled = currentPage >= totalPages;
      } else {
        pagination.style.display = 'none';
        currentPage = 1;
      }
      
      // Mostrar productos para la página actual
      const startIdx = (currentPage - 1) * itemsPerPage;
      const endIdx = startIdx + itemsPerPage;
      const productsToShow = encontrados.slice(startIdx, endIdx);
      
      let html = `<table>
        <thead>
          <tr>
            <th>Producto</th>
            <th>Cantidad</th>
            <th>Ubicación</th>
            <th>Obra</th>
            <th>Imagen</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>`;
      
      productsToShow.forEach(p => {
        html += `<tr>
          <td>${p.producto}</td>
          <td>${p.cantidad}</td>
          <td>Est: ${p.estanteria}, Bal: ${p.balda}</td>
          <td>${p.obra}</td>
          <td>${p.imagen ? `<img src="${p.imagen}" class="imagen-producto" style="max-width:80px;max-height:60px;">` : 'Sin imagen'}</td>
          <td>
            <button onclick="retirarPrompt('${p.producto}')">Retirar</button>
            <button class="secondary" onclick="aumentarPrompt('${p.producto}')">Aumentar</button>
          </td>
        </tr>`;
      });
      
      html += "</tbody></table>";
      div.innerHTML = html;
    }

    function changePage(delta) {
      const nombre = document.getElementById("productoConsulta").value.toLowerCase().trim();
      const ubicacion = document.getElementById("ubicacionConsulta").value.toLowerCase().trim();
      const obra = document.getElementById("obraConsulta").value.toLowerCase().trim();
      
      const encontrados = productos.filter(p => 
        (nombre === "" || p.producto.toLowerCase().includes(nombre)) &&
        (ubicacion === "" || 
          p.estanteria.toLowerCase().includes(ubicacion) || 
          p.balda.toLowerCase().includes(ubicacion)) &&
        (obra === "" || p.obra.toLowerCase() === obra)
      );
      
      const totalPages = Math.ceil(encontrados.length / itemsPerPage);
      currentPage += delta;
      
      if (currentPage < 1) currentPage = 1;
      if (currentPage > totalPages) currentPage = totalPages;
      
      consultarProducto();
    }

    function cargarObrasConsulta() {
      const select = document.getElementById("obraConsulta");
      select.innerHTML = '<option value="">Todas las obras</option>';
      
      const obrasUnicas = [...new Set(productos.map(p => p.obra))].filter(obra => obra);
      obrasUnicas.sort();
      
      obrasUnicas.forEach(obra => {
        const option = document.createElement("option");
        option.value = obra.toLowerCase();
        option.textContent = obra;
        select.appendChild(option);
      });
    }

    function retirarPrompt(prod) {
      const c = parseInt(prompt(`¿Cuántos retirar de '${prod}'?`));
      if (c > 0) retirarProducto(prod, c);
    }

    function retirarProducto(prodName, cant) {
      const p = productos.find(x => x.producto === prodName);
      if (p && p.cantidad >= cant) {
        p.cantidad -= cant;
        historial.push({
          tipo: "Retiro",
          fecha: new Date().toISOString().split('T')[0],
          usuario: document.getElementById("usuarioRegistro").value || "Anónimo",
          producto: prodName, cantidad: cant,
          estanteria: p.estanteria, balda: p.balda, obra: p.obra
        });
        guardarDatos();
        showNotification(`Retirado ${cant} de ${prodName}`);
        mostrarAlmacen();
        mostrarHistorial();
        consultarProducto();
      } else {
        showNotification("Cantidad insuficiente o producto no encontrado", "error");
      }
    }

    function aumentarPrompt(prod) {
      const c = parseInt(prompt(`¿Cuántos aumentar a '${prod}'?`));
      if (c > 0) aumentarProducto(prod, c);
    }

    function aumentarProducto(prodName, cant) {
      const p = productos.find(x => x.producto === prodName);
      if (p) {
        p.cantidad += cant;
        historial.push({
          tipo: "Aumento",
          fecha: new Date().toISOString().split('T')[0],
          usuario: document.getElementById("usuarioRegistro").value || "Anónimo",
          producto: prodName, cantidad: cant,
          estanteria: p.estanteria, balda: p.balda, obra: p.obra
        });
        guardarDatos();
        showNotification(`Aumentados ${cant} de ${prodName}`);
        mostrarAlmacen();
        mostrarHistorial();
        consultarProducto();
      } else {
        showNotification("No existe el producto", "error");
      }
    }

    function mostrarHistorial() {
      const div = document.getElementById("historialMovimientos");
      const filtro = document.getElementById("filtroHistorial").value.toLowerCase();
      const fechaDesde = document.getElementById("fechaDesdeHistorial").value;
      const fechaHasta = document.getElementById("fechaHastaHistorial").value;
      
      // Filtrar historial
      filteredHistorial = historial.filter(m => 
        (filtro === "" || 
          m.tipo.toLowerCase().includes(filtro) || 
          m.producto.toLowerCase().includes(filtro) ||
          m.usuario.toLowerCase().includes(filtro) ||
          m.obra.toLowerCase().includes(filtro)) &&
        (!fechaDesde || m.fecha >= fechaDesde) &&
        (!fechaHasta || m.fecha <= fechaHasta)
      );
      
      // Ordenar por fecha más reciente primero
      filteredHistorial.sort((a, b) => new Date(b.fecha) - new Date(a.fecha));
      
      // Paginación
      const totalPages = Math.ceil(filteredHistorial.length / itemsPerPage);
      const pagination = document.getElementById("paginationHistorial");
      
      if (filteredHistorial.length > itemsPerPage) {
        pagination.style.display = 'flex';
        document.getElementById("historialPageInfo").textContent = `Página ${currentHistorialPage} de ${totalPages}`;
        document.getElementById("prevHistorialPage").disabled = currentHistorialPage <= 1;
        document.getElementById("nextHistorialPage").disabled = currentHistorialPage >= totalPages;
      } else {
        pagination.style.display = 'none';
        currentHistorialPage = 1;
      }
      
      // Mostrar movimientos para la página actual
      const startIdx = (currentHistorialPage - 1) * itemsPerPage;
      const endIdx = startIdx + itemsPerPage;
      const movimientosToShow = filteredHistorial.slice(startIdx, endIdx);
      
      div.innerHTML = "<h3>Movimientos:</h3>";
      if (movimientosToShow.length === 0) {
        div.innerHTML += "<p>No se encontraron movimientos</p>";
        return;
      }
      
      let html = `<table>
        <thead>
          <tr>
            <th>Tipo</th>
            <th>Fecha</th>
            <th>Usuario</th>
            <th>Producto</th>
            <th>Cantidad</th>
            <th>Ubicación</th>
            <th>Obra</th>
          </tr>
        </thead>
        <tbody>`;
      
      movimientosToShow.forEach(m => {
        html += `<tr>
          <td>${m.tipo}</td>
          <td>${m.fecha}</td>
          <td>${m.usuario}</td>
          <td>${m.producto}</td>
          <td>${m.cantidad}</td>
          <td>${m.estanteria}, ${m.balda}</td>
          <td>${m.obra}</td>
        </tr>`;
      });
      
      html += "</tbody></table>";
      div.innerHTML += html;
    }

    function changeHistorialPage(delta) {
      const totalPages = Math.ceil(filteredHistorial.length / itemsPerPage);
      currentHistorialPage += delta;
      
      if (currentHistorialPage < 1) currentHistorialPage = 1;
      if (currentHistorialPage > totalPages) currentHistorialPage = totalPages;
      
      mostrarHistorial();
    }

    function eliminarProducto() {
      const val = document.getElementById("productoEliminar").value;
      if (!val) {
        showNotification("Selecciona un producto", "error");
        return;
      }
      
      productos = productos.filter(p => p.producto !== val);
      historial.push({
        tipo: "Eliminación",
        fecha: new Date().toISOString().split('T')[0],
        usuario: document.getElementById("usuarioRegistro").value || "Anónimo",
        producto: val, cantidad: "N/A",
        estanteria: "N/A", balda: "N/A", obra: "N/A"
      });
      guardarDatos();
      showNotification(`Producto ${val} eliminado`);
      mostrarAlmacen();
      mostrarHistorial();
      cargarSelects();
      cargarObrasConsulta();
    }

    function cargarSelects() {
      const selElim = document.getElementById("productoEliminar");
      const selEd = document.getElementById("productoEditar");
      selElim.innerHTML = "";
      selEd.innerHTML = "";
      
      // Ordenar productos alfabéticamente
      const productosOrdenados = [...productos].sort((a, b) => a.producto.localeCompare(b.producto));
      
      productosOrdenados.forEach(p => {
        const opt1 = document.createElement("option");
        opt1.value = p.producto;
        opt1.textContent = p.producto;
        selElim.appendChild(opt1);

        const opt2 = document.createElement("option");
        opt2.value = p.producto;
        opt2.textContent = p.producto;
        selEd.appendChild(opt2);
      });
    }

    function generarAlbaran() {
      const wb = XLSX.utils.book_new();
      const ws = XLSX.utils.json_to_sheet(historial);
      XLSX.utils.book_append_sheet(wb, ws, "Albaran");
      XLSX.writeFile(wb, `albaran_${new Date().toISOString().split('T')[0]}.xlsx`);
      showNotification("Albarán generado correctamente");
    }

    function enviarAlbaran() {
      showNotification("Albarán enviado (simulado)", "info");
    }

    function hacerBackup() {
      const data = JSON.stringify({ productos, historial });
      const blob = new Blob([data], { type: "application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = `backup_almacen_${new Date().toISOString().split('T')[0]}.json`;
      a.click();
      URL.revokeObjectURL(url);
      showNotification("Copia de seguridad generada");
    }

    function cargarBackup(e) {
      const file = e.target.files[0];
      if (!file) return;
      
      const reader = new FileReader();
      reader.onload = (evt) => {
        try {
          const data = JSON.parse(evt.target.result);
          productos = data.productos || [];
          historial = data.historial || [];
          guardarDatos();
          showNotification("Copia de seguridad cargada correctamente");
          mostrarAlmacen();
          mostrarHistorial();
          cargarSelects();
          cargarObrasConsulta();
        } catch (err) {
          showNotification("Error al cargar la copia: " + err.message, "error");
        }
      };
      reader.readAsText(file);
    }

    function guardarDatos() {
      localStorage.setItem("productos", JSON.stringify(productos));
      localStorage.setItem("historialMovimientos", JSON.stringify(historial));
    }

    // =============================
    // Funciones para gráficos estadísticos
    // =============================
    function actualizarGraficos() {
      const tipo = document.getElementById('tipoGrafico').value;
      
      // Destruir instancias anteriores
      if(mainChartInstance) mainChartInstance.destroy();
      if(secondaryChartInstance) secondaryChartInstance.destroy();

      switch(tipo) {
        case 'stock':
          crearGraficoBarrasStock();
          crearGraficoTopProductos();
          break;
        case 'ubicacion':
          crearGraficoUbicaciones();
          crearGraficoObras();
          break;
        case 'movimientos':
          crearGraficoMovimientos();
          crearGraficoTiposMovimientos();
          break;
      }
    }

    function crearGraficoBarrasStock() {
      const ctx = document.getElementById('mainChart').getContext('2d');
      const productosOrdenados = [...productos].sort((a, b) => b.cantidad - a.cantidad);

      mainChartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: productosOrdenados.map(p => p.producto),
          datasets: [{
            label: 'Stock Actual',
            data: productosOrdenados.map(p => p.cantidad),
            backgroundColor: 'rgba(54, 162, 235, 0.7)',
            borderColor: 'rgba(54, 162, 235, 1)',
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: { beginAtZero: true }
          },
          plugins: {
            title: { display: true, text: 'Stock por Producto' }
          }
        }
      });
    }

    function crearGraficoTopProductos() {
      const ctx = document.getElementById('secondaryChart').getContext('2d');
      const topProductos = [...productos].sort((a, b) => b.cantidad - a.cantidad).slice(0, 5);

      secondaryChartInstance = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: topProductos.map(p => p.producto),
          datasets: [{
            label: 'Distribución',
            data: topProductos.map(p => p.cantidad),
            backgroundColor: [
              'rgba(255, 99, 132, 0.7)',
              'rgba(75, 192, 192, 0.7)',
              'rgba(153, 102, 255, 0.7)',
              'rgba(255, 159, 64, 0.7)',
              'rgba(54, 162, 235, 0.7)'
            ]
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: { display: true, text: 'Top 5 Productos' },
            legend: { position: 'bottom' }
          }
        }
      });
    }

    function crearGraficoUbicaciones() {
      const ctx = document.getElementById('mainChart').getContext('2d');
      const ubicaciones = {};
      
      productos.forEach(p => {
        const ubicacion = `${p.estanteria}-${p.balda}`;
        ubicaciones[ubicacion] = (ubicaciones[ubicacion] || 0) + p.cantidad;
      });

      mainChartInstance = new Chart(ctx, {
        type: 'doughnut',
        data: {
          labels: Object.keys(ubicaciones),
          datasets: [{
            label: 'Stock por Ubicación',
            data: Object.values(ubicaciones),
            backgroundColor: Object.keys(ubicaciones).map((_, i) => 
              `hsl(${i * 360 / Object.keys(ubicaciones).length}, 70%, 50%)`)
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: { display: true, text: 'Distribución por Ubicación' }
          }
        }
      });
    }

    function crearGraficoObras() {
      const ctx = document.getElementById('secondaryChart').getContext('2d');
      const obras = {};
      
      productos.forEach(p => {
        obras[p.obra] = (obras[p.obra] || 0) + p.cantidad;
      });

      secondaryChartInstance = new Chart(ctx, {
        type: 'polarArea',
        data: {
          labels: Object.keys(obras),
          datasets: [{
            label: 'Stock por Obra',
            data: Object.values(obras),
            backgroundColor: Object.keys(obras).map((_, i) => 
              `hsl(${i * 360 / Object.keys(obras).length}, 70%, 50%)`)
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          plugins: {
            title: { display: true, text: 'Distribución por Obra' }
          }
        }
      });
    }

    function crearGraficoMovimientos() {
      const ctx = document.getElementById('mainChart').getContext('2d');
      const movimientosPorFecha = historial.reduce((acc, mov) => {
        acc[mov.fecha] = (acc[mov.fecha] || 0) + 1;
        return acc;
      }, {});

      const labels = Object.keys(movimientosPorFecha).sort();
      
      mainChartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Movimientos Diarios',
            data: labels.map(date => movimientosPorFecha[date]),
            borderColor: 'rgba(75, 192, 192, 1)',
            tension: 0.1
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: { beginAtZero: true }
          },
          plugins: {
            title: { display: true, text: 'Movimientos por Fecha' }
          }
        }
      });
    }

    function crearGraficoTiposMovimientos() {
      const ctx = document.getElementById('secondaryChart').getContext('2d');
      const tiposMovimientos = historial.reduce((acc, mov) => {
        acc[mov.tipo] = (acc[mov.tipo] || 0) + 1;
        return acc;
      }, {});

      secondaryChartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: Object.keys(tiposMovimientos),
          datasets: [{
            label: 'Cantidad de Movimientos',
            data: Object.values(tiposMovimientos),
            backgroundColor: 'rgba(255, 99, 132, 0.7)'
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: { beginAtZero: true }
          },
          plugins: {
            title: { display: true, text: 'Tipos de Movimientos' }
          }
        }
      });
    }

    function exportarChart(chartId) {
      const canvas = document.getElementById(chartId);
      const image = canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.download = `grafico_${chartId}_${new Date().toISOString().slice(0,10)}.png`;
      link.href = image;
      link.click();
    }

    async function exportarGraficos() {
      const container = document.querySelector('.charts-container');
      
      html2canvas(container).then(canvas => {
        const link = document.createElement('a');
        link.download = `reporte_almacen_${new Date().toISOString().slice(0,10)}.png`;
        link.href = canvas.toDataURL('image/png');
        link.click();
      });
    }

    // =============================
    // Login
    // =============================
    document.getElementById("loginForm").addEventListener("submit", (e) => {
      e.preventDefault();
      const user = document.getElementById("usuario").value;
      const pass = document.getElementById("password").value;
      
      // Validación simple (en producción usar autenticación segura)
      if (user === "admin" && pass === "admin123") {
        document.getElementById("loginScreen").style.display = "none";
        document.getElementById("tabs").style.display = "block";
        document.getElementById("usuarioRegistro").value = user;
        mostrarAlmacen();
        mostrarHistorial();
        cargarSelects();
        cargarObrasConsulta();
      } else {
        document.getElementById("error-message").style.display = "block";
      }
    });

    // =============================
    // Event listeners adicionales
    // =============================
    document.getElementById("filtroAlmacen").addEventListener("input", mostrarAlmacen);
    document.getElementById("ordenAlmacen").addEventListener("change", mostrarAlmacen);
    document.getElementById("filtroHistorial").addEventListener("input", mostrarHistorial);
    document.getElementById("fechaDesdeHistorial").addEventListener("change", mostrarHistorial);
    document.getElementById("fechaHastaHistorial").addEventListener("change", mostrarHistorial);

    // =============================
    // Inicialización
    // =============================
    document.addEventListener("DOMContentLoaded", function() {
      document.getElementById("fecha").value = new Date().toISOString().split('T')[0];
      document.getElementById("fechaDesdeHistorial").value = new Date(new Date().setMonth(new Date().getMonth() - 1)).toISOString().split('T')[0];
      document.getElementById("fechaHastaHistorial").value = new Date().toISOString().split('T')[0];
    });
 <script>
  // Configuración Firebase (reemplaza con tus datos)
  const firebaseConfig = {
    apiKey: "TU_API_KEY",
    authDomain: "TU_PROYECTO.firebaseapp.com",
    projectId: "TU_PROYECTO",
    storageBucket: "TU_PROYECTO.appspot.com",
    messagingSenderId: "NUMERO_UNICO",
    appId: "APP_ID"
  };

  // Inicializa Firebase
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();
  const auth = firebase.auth();

  // Modifica estas funciones para usar Firestore:
  function guardarDatos() {
    // Ejemplo para productos:
    db.collection("productos").doc("lista").set({
      items: productos
    });
  }

  function cargarDatos() {
    db.collection("productos").doc("lista").get().then((doc) => {
      if (doc.exists) {
        productos = doc.data().items || [];
      }
    });
  }
</script>
