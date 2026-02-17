[index.html](https://github.com/user-attachments/files/25353762/index.html)
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Farmacia DPSST</title>
    <link rel="icon" href="data:,">
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2.39.3/dist/umd/supabase.js"></script>
    <style>
        :root { --primary: #2c3e50; --secondary: #27ae60; --danger: #e74c3c; --warning: #f39c12; --accent: #8e44ad; --light: #f8f9fa; }
        body { font-family: 'Segoe UI', sans-serif; background: #eef2f3; margin: 0; padding: 10px; }
        .container { max-width: 1000px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        .panel { background: var(--light); padding: 15px; border-radius: 10px; margin-bottom: 15px; border: 1px solid #ddd; }
        .grid-inventario { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 15px; margin: 20px 0; }
        .card-producto { background: white; padding: 15px; border-radius: 10px; border: 1px solid #eee; text-align: center; cursor: pointer; transition: 0.3s; }
        .card-producto:hover { transform: translateY(-5px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .stock-numero { display: block; font-size: 2em; font-weight: bold; margin: 10px 0; }
        .flex-row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
        input, select, button { padding: 12px; border: 1px solid #ccc; border-radius: 8px; font-size: 14px; }
        .btn-add { background: var(--secondary); color: white; border: none; font-weight: bold; cursor: pointer; flex-grow: 1; }
        .btn-jefa { background: var(--warning); border: none; font-weight: bold; cursor: pointer; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th { background: var(--primary); color: white; padding: 12px; text-align: left; }
        td { padding: 12px; border-bottom: 1px solid #eee; text-transform: capitalize; }
        @keyframes parpadeo { 0%, 100% { background: #ffdada; } 50% { background: #ff4d4d; color: white; } }
        .alerta-cierre { animation: parpadeo 1.5s infinite; border: 2px solid var(--danger) !important; }
    </style>
</head>
<body>

<div class="container">
    <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
        <h2 style="margin:0; color: var(--primary);">🏥 Farmacia DPSST</h2>
        <button class="btn-jefa" onclick="accesoJefa()">🔑 Admin Jefa</button>
    </div>

    <div id="status-global" style="padding:15px; border-radius:8px; text-align:center; font-weight:bold; margin-bottom:15px;">Iniciando conexión...</div>

    <div id="panel-jefa" class="panel" style="display:none; border: 2px solid var(--accent);">
        <h3 style="color: var(--accent); margin-top:0;">🛠 Configuración Maestra</h3>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom:15px;">
            <div style="background:white; padding:10px; border-radius:8px; border:1px solid #ddd;">
                <strong>➕ Nueva Enfermera</strong>
                <div class="flex-row" style="margin-top:5px;">
                    <input type="text" id="nuevo-nombre" placeholder="Nombre..." style="flex-grow:1;">
                    <button onclick="guardarNuevaEnfermera()" style="background:var(--accent); color:white; border:none;">Ok</button>
                </div>
                <div id="lista-gestion-personal" style="margin-top:10px; font-size:12px; max-height:80px; overflow-y:auto;"></div>
            </div>
            <div style="background:white; padding:10px; border-radius:8px; border:1px solid #ddd;">
                <strong>🧨 Peligro</strong><br>
                <button onclick="resetearSistema()" style="background:black; color:white; border:none; width:100%; margin-top:5px; cursor:pointer; padding:10px;">BORRAR TODO EL INVENTARIO</button>
            </div>
        </div>
        <button onclick="document.getElementById('panel-jefa').style.display='none'" style="width:100%;">Cerrar Panel</button>
    </div>

    <div class="panel">
        <div class="flex-row">
            <strong>👩‍⚕️ Enfermera:</strong>
            <select id="select-enfermera" style="flex-grow:1;" onchange="tomarTurno()"></select>
        </div>
    </div>

    <div class="panel">
        <h3 style="margin-top:0;">📦 Stock Consolidado</h3>
        <div id="lista-inventario" class="grid-inventario"></div>
    </div>

    <div id="seccion-registro" class="panel" style="opacity:0.5; pointer-events:none;">
        <div class="flex-row">
            <input type="text" id="nombre-prod" placeholder="Nombre del producto..." style="flex-grow:2;" oninput="filtrarTarjetas()">
            <input type="number" id="cant-prod" placeholder="Cant." style="width:80px;">
            <select id="tipo-mov"><option value="Salida">Salida (-)</option><option value="Entrada">Entrada (+)</option></select>
            <button class="btn-add" onclick="registrarMovimiento()">REGISTRAR</button>
        </div>
    </div>

    <table>
        <thead><tr><th>Hora</th><th>Producto</th><th>Tipo</th><th>Cant.</th><th>Responsable</th></tr></thead>
        <tbody id="tabla-body"></tbody>
    </table>
</div>

<script>
    // REEMPLAZAR CON TUS DATOS DE SUPABASE
    const SUPABASE_URL = 'https://iqaagncrmcylemuwhlpr.supabase.co';
    const SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImlxYWFnbmNybWN5bGVtdXdobHByIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzExNzcwOTgsImV4cCI6MjA4Njc1MzA5OH0.DwF3bR_Q1VwlM-19Z6w1mkmw0SnpelnYW_uVwxc3TKw';
    const supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
    const CLAVE_MAESTRA = "jefa123";

    let responsableActiva = "";
    let inventarioLocal = [];

    async function iniciar() {
        try {
            await cargarListaEnfermeras();
            await cargarEstadoGlobal();
            await cargarInventario();
            await cargarMovimientos();
            setInterval(cargarEstadoGlobal, 60000);
            supabaseClient.channel('cambios').on('postgres_changes', { event: '*', schema: 'public' }, () => {
                cargarEstadoGlobal(); cargarInventario(); cargarMovimientos();
            }).subscribe();
        } catch(e) { console.error(e); }
    }

    async function registrarMovimiento() {
        const n = document.getElementById('nombre-prod').value.trim().toLowerCase();
        const c = parseInt(document.getElementById('cant-prod').value);
        const t = document.getElementById('tipo-mov').value;
        if (!n || !c || c <= 0) return;

        const prod = inventarioLocal.find(p => p.nombre.toLowerCase() === n);
        if (t === 'Salida' && (!prod || prod.stock_total < c)) return alert("❌ No hay stock suficiente o el producto no existe.");

        await supabaseClient.from('movimientos').insert([{
            nombre: n, cantidad: c, tipo: t, responsable: responsableActiva, turno_id: 'actual'
        }]);

        document.getElementById('nombre-prod').value = "";
        document.getElementById('cant-prod').value = "";
        document.getElementById('nombre-prod').focus();
    }

    async function resetearSistema() {
        if (confirm("⚠️ ¿BORRAR TODO? Se perderá el rastro de Rukia, Byakuya y Orihime.") && prompt("Escribe: BORRAR") === "BORRAR") {
            await supabaseClient.from('movimientos').delete().neq('id', 0);
            alert("Sistema reiniciado."); location.reload();
        }
    }

    async function cargarListaEnfermeras() {
        const { data } = await supabaseClient.from('personal').select('*').order('nombre');
        const sel = document.getElementById('select-enfermera');
        const listG = document.getElementById('lista-gestion-personal');
        sel.innerHTML = '<option value="">-- Seleccionar --</option>';
        listG.innerHTML = '';
        data?.forEach(p => {
            sel.innerHTML += `<option value="${p.nombre.toLowerCase()}">${p.nombre}</option>`;
            listG.innerHTML += `<div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee;">${p.nombre} <button onclick="borrarP(${p.id})" style="color:red; border:none; background:none; cursor:pointer;">✕</button></div>`;
        });
    }

    async function cargarEstadoGlobal() {
        const { data } = await supabaseClient.from('configuracion').select('*').eq('id', 'sistema_control').single();
        const st = document.getElementById('status-global');
        const reg = document.getElementById('seccion-registro');
        const ahora = new Date();
        if(data?.estado_bloqueo) {
            let corte = new Date(data.fecha_inicio); corte.setDate(corte.getDate()+1); corte.setHours(8,0,0,0);
            const min = Math.floor((corte - ahora)/60000);
            if(ahora >= corte) { await cerrarTurno(); return; }
            st.innerText = min <= 30 ? `⚠️ CIERRE EN ${min} MIN` : `🔒 GUARDIA: ${data.valor_texto}`;
            st.className = min <= 30 ? "alerta-cierre" : "";
            st.style.background = min <= 30 ? "" : "#ffdada";
            reg.style.opacity = "1"; reg.style.pointerEvents = "auto";
        } else {
            st.innerText = "🔓 ESPERANDO NUEVO TURNO"; st.style.background = "#d4edda";
            reg.style.opacity = "0.5"; reg.style.pointerEvents = "none";
        }
        responsableActiva = data?.valor_texto;
        document.getElementById('select-enfermera').disabled = data?.estado_bloqueo;
    }

    async function cargarInventario() {
        const { data } = await supabaseClient.from('inventario_actual').select('*').order('nombre');
        inventarioLocal = data || [];
        const cont = document.getElementById('lista-inventario');
        cont.innerHTML = '';
        inventarioLocal.forEach(i => {
            const col = i.stock_total <= 0 ? 'var(--danger)' : (i.stock_total < 10 ? 'var(--warning)' : 'var(--secondary)');
            cont.innerHTML += `<div class="card-producto" style="border-top:5px solid ${col}" onclick="selP('${i.nombre}')"><strong>${i.nombre}</strong><span class="stock-numero" style="color:${col}">${i.stock_total}</span></div>`;
        });
    }

    async function cargarMovimientos() {
        const { data } = await supabaseClient.from('movimientos').select('*').eq('turno_id', 'actual').order('id', {ascending:false});
        const b = document.getElementById('tabla-body'); b.innerHTML = '';
        data?.forEach(r => b.innerHTML += `<tr><td>${new Date(r.fecha).toLocaleTimeString([], {hour:'2-digit', minute:'2-digit'})}</td><td>${r.nombre}</td><td>${r.tipo}</td><td>${r.cantidad}</td><td>${r.responsable}</td></tr>`);
    }

    function selP(n) { document.getElementById('nombre-prod').value = n; document.getElementById('cant-prod').focus(); }
    function filtrarTarjetas() {
        const b = document.getElementById('nombre-prod').value.toLowerCase();
        document.querySelectorAll('.card-producto').forEach(t => t.style.display = t.innerText.toLowerCase().includes(b) ? "block" : "none");
    }
    async function accesoJefa() { if(prompt("CLAVE:") === CLAVE_MAESTRA) document.getElementById('panel-jefa').style.display='block'; }
    async function guardarNuevaEnfermera() {
        const n = document.getElementById('nuevo-nombre').value.trim();
        if(n) { await supabaseClient.from('personal').insert([{nombre:n}]); document.getElementById('nuevo-nombre').value=""; cargarListaEnfermeras(); }
    }
    async function borrarP(id) { if(confirm("¿Borrar?")) { await supabaseClient.from('personal').delete().eq('id',id); cargarListaEnfermeras(); } }
    async function tomarTurno() {
        const n = document.getElementById('select-enfermera').value;
        if(n && confirm(`¿Iniciar guardia?`)) await supabaseClient.from('configuracion').update({ valor_texto:n, estado_bloqueo:true, fecha_inicio:new Date().toISOString() }).eq('id','sistema_control');
    }
    async function cerrarTurno() {
        await supabaseClient.from('movimientos').update({ turno_id:'cerrado-'+Date.now() }).eq('turno_id','actual');
        await supabaseClient.from('configuracion').update({ estado_bloqueo:false, valor_texto:"", fecha_inicio:null }).eq('id','sistema_control');
        location.reload();
    }

    iniciar();
</script>
</body>
</html>
