          </div>
          <div class="stat-card" id="highTasks">
            <span class="stat-number">0</span>
            <span class="stat-label">Alta Prioridad</span>
          </div>
          <div class="stat-card" id="pendingTasks">
            <span class="stat-number">0</span>
            <span class="stat-label">Pendientes</span>
          </div>
        </div>
      </div>

      <!-- Sección del Formulario y Lista -->
      <div class="form-section">
        <h3 class="section-title"><i class="fas fa-plus-circle"></i> Nuevo Encargo</h3>
        
        <form id="taskForm">
          <div class="form-group">
            <label for="clientName">Nombre del cliente</label>
            <input type="text" id="clientName" placeholder="Ej: María García" required>
          </div>
          
          <div class="form-group">
            <label for="taskDescription">Descripción del encargo</label>
            <textarea id="taskDescription" rows="3" placeholder="Ej: Bolso gris con asas largas y detalles en blanco"></textarea>
          </div>
          
          <div class="form-group">
            <label for="dueDate">Fecha de entrega</label>
            <input type="date" id="dueDate" required>
          </div>
          
          <div class="form-group">
            <label for="priority">Prioridad</label>
            <select id="priority">
              <option value="low">🟢 Baja</option>
              <option value="medium">🟡 Media</option>
              <option value="high">🔴 Alta</option>
            </select>
          </div>

          <button type="submit" class="btn btn-primary">
            <i class="fas fa-save"></i> Guardar Encargo
          </button>
        </form>
        
        <div class="filter-buttons">
          <button class="filter-btn active" data-filter="all">Todos</button>
          <button class="filter-btn" data-filter="high">Alta</button>
          <button class="filter-btn" data-filter="pending">Pendientes</button>
        </div>
        
        <h3 class="section-title"><i class="fas fa-list"></i> Lista de Encargos</h3>
        <input type="text" id="searchInput" placeholder="🔍 Buscar encargos..." class="form-group">
        
        <div class="task-list" id="taskList">
          <div class="empty-state">
            <i class="fas fa-clipboard-list"></i>
            <p>No hay encargos registrados</p>
          </div>
        </div>
        
        <div class="btn-group">
          <button id="exportBtn" class="btn btn-warning">
            <i class="fas fa-file-pdf"></i> Exportar a PDF
          </button>
          <button id="statsBtn" class="btn btn-secondary">
            <i class="fas fa-chart-pie"></i> Ver Estadísticas
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal de Estadísticas -->
  <div id="statsModal" class="modal">
    <div class="modal-content">
      <button class="close" id="closeStats">&times;</button>
      <h2><i class="fas fa-chart-pie"></i> Estadísticas de Encargos</h2>
      <div id="statsContent">
        <!-- El contenido de estadísticas se generará aquí -->
      </div>
    </div>
  </div>

  <script>
    // Variables globales
    let orders = [];
    let currentDate = new Date();
    let currentFilter = 'all';
    let editingIndex = -1;

    // Inicialización
    document.addEventListener('DOMContentLoaded', function() {
      loadOrders();
      renderCalendar();
      updateStats();
      renderOrderList();
      setupEventListeners();
      
      // Establecer fecha mínima como hoy
      const today = new Date().toISOString().split('T')[0];
      document.getElementById('dueDate').min = today;
      document.getElementById('dueDate').value = today;
    });

    // Cargar órdenes desde localStorage
    function loadOrders() {
      const savedOrders = localStorage.getItem('yulissa_orders');
      if (savedOrders) {
        orders = JSON.parse(savedOrders);
      }
    }

    // Guardar órdenes en localStorage
    function saveOrders() {
      localStorage.setItem('yulissa_orders', JSON.stringify(orders));
      renderCalendar();
      updateStats();
      renderOrderList();
    }

    // Configurar event listeners
    function setupEventListeners() {
      // Navegación del calendario
      document.getElementById('prevBtn').addEventListener('click', () => {
        currentDate.setMonth(currentDate.getMonth() - 1);
        renderCalendar();
      });
      
      document.getElementById('nextBtn').addEventListener('click', () => {
        currentDate.setMonth(currentDate.getMonth() + 1);
        renderCalendar();
      });
      
      // Formulario
      document.getElementById('taskForm').addEventListener('submit', function(e) {
        e.preventDefault();
        saveOrder();
      });
      
      // Filtros
      document.querySelectorAll('.filter-btn').forEach(btn => {
        btn.addEventListener('click', function() {
          document.querySelectorAll('.filter-btn').forEach(b => b.classList.remove('active'));
          this.classList.add('active');
          currentFilter = this.getAttribute('data-filter');
          renderOrderList();
        });
      });
      
      // Búsqueda
      document.getElementById('searchInput').addEventListener('input', renderOrderList);
      
      // Botones de exportación y estadísticas
      document.getElementById('exportBtn').addEventListener('click', exportToPDF);
      document.getElementById('statsBtn').addEventListener('click', showStats);
      document.getElementById('closeStats').addEventListener('click', () => {
        document.getElementById('statsModal').style.display = 'none';
      });
      
      // Cerrar modal al hacer clic fuera
      window.addEventListener('click', function(e) {
        if (e.target === document.getElementById('statsModal')) {
          document.getElementById('statsModal').style.display = 'none';
        }
      });
    }

    // Renderizar calendario
    function renderCalendar() {
      const calendarBody = document.getElementById('calendar-body');
      const monthYear = document.getElementById('monthYear');
      
      // Limpiar calendario
      calendarBody.innerHTML = '';
      
      // Establecer texto del mes y año
      const monthNames = ['Enero', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 
                         'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'];
      monthYear.textContent = `${monthNames[currentDate.getMonth()]} ${currentDate.getFullYear()}`;
      
      // Obtener primer día del mes y cantidad de días
      const firstDay = new Date(currentDate.getFullYear(), currentDate.getMonth(), 1);
      const lastDay = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 0);
      const daysInMonth = lastDay.getDate();
      const startingDay = firstDay.getDay(); // 0 (domingo) a 6 (sábado)
      
      // Crear calendario
      let date = 1;
      for (let i = 0; i < 6; i++) {
        const row = document.createElement('tr');
        
        for (let j = 0; j < 7; j++) {
          const cell = document.createElement('td');
          
          if (i === 0 && j < startingDay) {
            // Celda vacía antes del primer día del mes
            cell.textContent = '';
          } else if (date > daysInMonth) {
            // Celda vacía después del último día del mes
            break;
          } else {
            // Celda con fecha
            cell.textContent = date;
            
            // Comprobar si es hoy
            const today = new Date();
            if (currentDate.getMonth() === today.getMonth() && 
                currentDate.getFullYear() === today.getFullYear() && 
                date === today.getDate()) {
              cell.classList.add('today');
            }
            
            // Comprobar si hay órdenes para esta fecha
            const cellDate = new Date(currentDate.getFullYear(), currentDate.getMonth(), date);
            const dateString = formatDate(cellDate);
            const dayOrders = orders.filter(order => order.dueDate === dateString);
            
            if (dayOrders.length > 0) {
              cell.classList.add('has-task');
              
              // Añadir indicador de prioridad más alta
              const hasHigh = dayOrders.some(order => order.priority === 'high');
              const hasMedium = dayOrders.some(order => order.priority === 'medium');
              
              if (hasHigh) {
                cell.classList.add('priority-high');
              } else if (hasMedium) {
                cell.classList.add('priority-medium');
              } else {
                cell.classList.add('priority-low');
              }
              
              // Añadir tooltip con información de órdenes
              cell.title = `${dayOrders.length} encargo(s) este día`;
              
              // Añadir evento para arrastrar y soltar
              cell.addEventListener('dragover', allowDrop);
              cell.addEventListener('drop', (e) => dropOrder(e, dateString));
            }
            
            date++;
          }
          
          row.appendChild(cell);
        }
        
        calendarBody.appendChild(row);
        
        if (date > daysInMonth) {
          break;
        }
      }
    }

    // Guardar una nueva orden o editar existente
    function saveOrder() {
      const clientName = document.getElementById('clientName').value.trim();
      const taskDescription = document.getElementById('taskDescription').value.trim();
      const dueDate = document.getElementById('dueDate').value;
      const priority = document.getElementById('priority').value;
      
      if (!clientName || !dueDate) {
        alert('Por favor, complete todos los campos obligatorios.');
        return;
      }
      
      const newOrder = {
        clientName,
        taskDescription,
        dueDate,
        priority,
        createdAt: new Date().toISOString()
      };
      
      if (editingIndex === -1) {
        // Nueva orden
        orders.push(newOrder);
      } else {
        // Editar orden existente
        orders[editingIndex] = newOrder;
        editingIndex = -1;
      }
      
      saveOrders();
      document.getElementById('taskForm').reset();
      
      // Restablecer la fecha a hoy
      const today = new Date().toISOString().split('T')[0];
      document.getElementById('dueDate').value = today;
    }

    // Renderizar lista de órdenes
    function renderOrderList() {
      const taskList = document.getElementById('taskList');
      const searchTerm = document.getElementById('searchInput').value.toLowerCase();
      
      // Filtrar órdenes según criterios
      let filteredOrders = orders;
      
      // Aplicar filtro
      if (currentFilter === 'high') {
        filteredOrders = filteredOrders.filter(order => order.priority === 'high');
      } else if (currentFilter === 'pending') {
        const today = new Date().toISOString().split('T')[0];
        filteredOrders = filteredOrders.filter(order => order.dueDate < today);
      }
      
      // Aplicar búsqueda
      if (searchTerm) {
        filteredOrders = filteredOrders.filter(order => 
          order.clientName.toLowerCase().includes(searchTerm) || 
          order.taskDescription.toLowerCase().includes(searchTerm)
        );
      }
      
      // Limpiar lista
      taskList.innerHTML = '';
      
      if (filteredOrders.length === 0) {
        taskList.innerHTML = `
          <div class="empty-state">
            <i class="fas fa-clipboard-list"></i>
            <p>No hay encargos que mostrar</p>
          </div>
        `;
        return;
      }
      
      // Agregar órdenes a la lista
      filteredOrders.forEach((order, index) => {
        const orderElement = document.createElement('div');
        orderElement.className = 'task-item';
        orderElement.draggable = true;
        orderElement.setAttribute('data-index', orders.indexOf(order));
        
        orderElement.innerHTML = `
          <div class="task-content">
            <div class="task-text"><strong>${escapeHtml(order.clientName)}</strong>: ${escapeHtml(order.taskDescription)}</div>
            <div class="task-date">Entrega: ${formatDisplayDate(order.dueDate)} - 
              ${order.priority === 'high' ? '🔴 Alta' : order.priority === 'medium' ? '🟡 Media' : '🟢 Baja'} Prioridad
            </div>
          </div>
          <div class="task-actions">
            <button class="btn btn-secondary btn-small edit-btn">
              <i class="fas fa-edit"></i> Editar
            </button>
            <button class="btn btn-danger btn-small delete-btn">
              <i class="fas fa-trash"></i> Eliminar
            </button>
          </div>
        `;
        
        // Añadir eventos para arrastrar y soltar
        orderElement.addEventListener('dragstart', dragOrder);
        
        // Añadir eventos para editar y eliminar
        orderElement.querySelector('.edit-btn').addEventListener('click', () => editOrder(orders.indexOf(order)));
        orderElement.querySelector('.delete-btn').addEventListener('click', () => deleteOrder(orders.indexOf(order)));
        
        taskList.appendChild(orderElement);
      });
    }

    // Editar orden
    function editOrder(index) {
      const order = orders[index];
      document.getElementById('clientName').value = order.clientName;
      document.getElementById('taskDescription').value = order.taskDescription;
      document.getElementById('dueDate').value = order.dueDate;
      document.getElementById('priority').value = order.priority;
      
      editingIndex = index;
      
      // Scroll al formulario
      document.getElementById('taskForm').scrollIntoView({ behavior: 'smooth' });
    }

    // Eliminar orden
    function deleteOrder(index) {
      if (confirm('¿Estás seguro de que deseas eliminar este encargo?')) {
        orders.splice(index, 1);
        saveOrders();
      }
    }

    // Funciones para arrastrar y soltar
    function allowDrop(e) {
      e.preventDefault();
    }

    function dragOrder(e) {
      e.dataTransfer.setData("text/plain", e.target.getAttribute('data-index'));
    }

    function dropOrder(e, dateString) {
      e.preventDefault();
      const orderIndex = e.dataTransfer.getData("text/plain");
      
      if (orderIndex) {
        orders[orderIndex].dueDate = dateString;
        saveOrders();
        
        // Efecto visual de confirmación
        e.target.classList.add('priority-medium');
        setTimeout(() => {
          e.target.classList.remove('priority-medium');
        }, 1000);
      }
    }

    // Actualizar estadísticas
    function updateStats() {
      const total = orders.length;
      const highPriority = orders.filter(order => order.priority === 'high').length;
      
      const today = new Date().toISOString().split('T')[0];
      const pending = orders.filter(order => order.dueDate < today).length;
      
      document.getElementById('totalTasks').querySelector('.stat-number').textContent = total;
      document.getElementById('highTasks').querySelector('.stat-number').textContent = highPriority;
      document.getElementById('pendingTasks').querySelector('.stat-number').textContent = pending;
    }

    // Mostrar estadísticas en modal
    function showStats() {
      const statsContent = document.getElementById('statsContent');
      const total = orders.length;
      const highPriority = orders.filter(order => order.priority === 'high').length;
      const mediumPriority = orders.filter(order => order.priority === 'medium').length;
      const lowPriority = orders.filter(order => order.priority === 'low').length;
      
      const today = new Date().toISOString().split('T')[0];
      const pending = orders.filter(order => order.dueDate < today).length;
      const completed = orders.filter(order => order.dueDate >= today).length;
      
      statsContent.innerHTML = `
        <div style="display: grid; grid-template-columns: 1fr; gap: 15px; margin: 20px 0;">
          <div style="background: #f0f7ff; padding: 15px; border-radius: 12px;">
            <h3 style="color: var(--secondary); margin-bottom: 10px;">Resumen General</h3>
            <p>Total de encargos: <strong>${total}</strong></p>
            <p>Encargos pendientes: <strong>${pending}</strong></p>
            <p>Encargos en tiempo: <strong>${completed}</strong></p>
          </div>
          
          <div style="background: #f0f7ff; padding: 15px; border-radius: 12px;">
            <h3 style="color: var(--secondary); margin-bottom: 10px;">Por Prioridad</h3>
            <p>🔴 Alta prioridad: <strong>${highPriority}</strong></p>
            <p>🟡 Media prioridad: <strong>${mediumPriority}</strong></p>
            <p>🟢 Baja prioridad: <strong>${lowPriority}</strong></p>
          </div>
        </div>
        
        <h3 style="color: var(--secondary); margin: 20px 0 10px;">Próximos vencimientos</h3>
        <div style="max-height: 200px; overflow-y: auto;">
          ${getUpcomingOrdersList()}
        </div>
      `;
      
      document.getElementById('statsModal').style.display = 'flex';
    }

    // Obtener lista de órdenes próximas a vencer
    function getUpcomingOrdersList() {
      const today = new Date();
      const nextWeek = new Date();
      nextWeek.setDate(today.getDate() + 7);
      
      const upcomingOrders = orders.filter(order => {
        const dueDate = new Date(order.dueDate);
        return dueDate >= today && dueDate <= nextWeek;
      }).sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
      
      if (upcomingOrders.length === 0) {
        return '<p>No hay encargos próximos a vencer en la próxima semana.</p>';
      }
      
      let html = '';
      upcomingOrders.forEach(order => {
        html += `
          <div style="padding: 10px; border-bottom: 1px solid #eee;">
            <strong>${escapeHtml(order.clientName)}</strong> - ${escapeHtml(order.taskDescription)}<br>
            <span style="font-size: 0.9em; color: #666;">Vence: ${formatDisplayDate(order.dueDate)}</span>
          </div>
        `;
      });
      
      return html;
    }

    // Exportar a PDF
    function exportToPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      
      // Título
      doc.setFontSize(20);
      doc.setTextColor(54, 209, 220);
      doc.text('Encargos - Yulissa Crochet', 105, 15, { align: 'center' });
      doc.setFontSize(12);
      doc.setTextColor(100, 100, 100);
      doc.text(`Generado el: ${new Date().toLocaleDateString('es-ES')}`, 105, 22, { align: 'center' });
      
      // Contenido
      let y = 35;
      doc.setFontSize(12);
      doc.setTextColor(0, 0, 0);
      
      orders.forEach((order, index) => {
        if (y > 270) {
          doc.addPage();
          y = 20;
        }
        
        // Nombre del cliente y descripción
        doc.setFont(undefined, 'bold');
        doc.text(`${index + 1}. ${order.clientName}:`, 15, y);
        doc.setFont(undefined, 'normal');
        
        // Descripción (con salto de línea si es muy larga)
        const description = doc.splitTextToSize(order.taskDescription, 160);
        doc.text(description, 25, y + 7);
        
        // Fecha y prioridad
        doc.setFontSize(10);
        doc.text(`Fecha de entrega: ${formatDisplayDate(order.dueDate)} | Prioridad: ${getPriorityText(order.priority)}`, 15, y + 7 + (description.length * 5));
        
        // Aumentar Y según la cantidad de líneas de descripción
        y += 15 + (description.length * 5);
      });
      
      // Guardar PDF
      doc.save(`encargos-yulissa-${new Date().toISOString().split('T')[0]}.pdf`);
    }

    // Funciones de utilidad para formato
    function formatDate(date) {
      return date.toISOString().split('T')[0];
    }

    function formatDisplayDate(dateString) {
      const options = { day: '2-digit', month: '2-digit', year: 'numeric' };
      return new Date(dateString).toLocaleDateString('es-ES', options);
    }

    function getPriorityText(priority) {
      return priority === 'high' ? 'Alta' : priority === 'medium' ? 'Media' : 'Baja';
    }

    // Escapar HTML para evitar problemas con caracteres especiales
    function escapeHtml(unsafe) {
      if (!unsafe) return '';
      return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
    }
  </script>
</body>
</html>
