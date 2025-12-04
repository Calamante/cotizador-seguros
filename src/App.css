import React, { useState, useEffect, useCallback, useMemo } from 'react';
// Íconos de lucide-react 
import { Home, User, DollarSign, Calculator, ChevronRight, AlertCircle, RefreshCw, Save, List, CheckCircle, XCircle, Info } from 'lucide-react';

// --- CONFIGURACIÓN Y DATOS INICIALES ---
const VALOR_BASE_M2 = 500; 
const FACTORES = { 
  ubicacion: { 'Ciudad Capital': 1.2, 'Zona Residencial': 1.1, 'Pueblo Pequeño': 0.95, },
  tipoPropiedad: { 'Casa': 1.0, 'Departamento': 0.9, 'Chalet/Villa': 1.3, 'Duplex': 1.05, },
  antiguedad: { 'Nueva (0-5 años)': 1.0, 'Media (6-20 años)': 1.1, 'Antigua (+20 años)': 1.25, },
};
const COBERTURAS_BASE = [
  { id: 'incendio', nombre: 'Incendio y Daños Estructurales', costoFactor: 0.05, descripcion: 'Cubre daños causados por fuego, humo y explosiones.' },
  { id: 'robo', nombre: 'Robo y Contenido', costoFactor: 0.08, descripcion: 'Protección contra el hurto de bienes dentro de la propiedad.' },
  { id: 'desastres', nombre: 'Desastres Naturales (Terremoto/Inundación)', costoFactor: 0.12, descripcion: 'Cubre eventos climáticos y geológicos mayores.' },
  { id: 'responsabilidad', nombre: 'Responsabilidad Civil', costoFactor: 0.03, descripcion: 'Protección por daños a terceros dentro o fuera de la propiedad.' },
];

const ESTADO_INICIAL = {
  nombre: '', edad: '', m2: '', 
  pais: 'Argentina', provincia: '', ciudad: '', direccion: '',
  ubicacion: '', tipoPropiedad: '', antiguedad: '',
  coberturasSeleccionadas: [],
};

// Clave para localStorage
const STORAGE_KEY = 'cotizaciones_seguros_data';

// --- FUNCIONES DE PERSISTENCIA (localStorage) ---
const loadFromLocalStorage = () => {
    try {
        const data = localStorage.getItem(STORAGE_KEY);
        // Deserializar las fechas de guardado si existen
        const parsedData = data ? JSON.parse(data) : [];
        return parsedData.map(item => ({
            ...item,
            // Asegura que las fechas sean objetos Date al cargar
            fechaGuardado: item.fechaGuardado ? new Date(item.fechaGuardado) : new Date(),
            fechaContratacion: item.fechaContratacion ? new Date(item.fechaContratacion) : undefined,
            fechaCancelacion: item.fechaCancelacion ? new Date(item.fechaCancelacion) : undefined,
        }));
    } catch (e) {
        console.error("Error loading from localStorage", e);
        return [];
    }
};

const saveToLocalStorage = (data) => {
    try {
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
    } catch (e) {
        console.error("Error saving to localStorage", e);
    }
};

// --- FUNCIONES DE FORMATO ESTABLES ---
const formatCurrency = (amount) => new Intl.NumberFormat('es-AR', { style: 'currency', currency: 'ARS', minimumFractionDigits: 2 }).format(amount);
const formatDate = (date) => date instanceof Date && !isNaN(date) ? date.toLocaleDateString('es-AR') : 'N/A';
const formatPercent = (factor) => factor > 1 ? `+${((factor - 1) * 100).toFixed(0)}%` : `${(factor * 100).toFixed(0)}%`;


// --- COMPONENTES AUXILIARES ---

// Componente para campos de texto (Bootstrap style)
const Input = React.memo(({ name, label, type = 'text', value, onChange, errors, min, disabled = false, placeholder = '' }) => (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">{label}</label>
      <input 
        type={type} 
        name={name} 
        id={name} 
        value={value} 
        onChange={onChange} 
        className={`form-control ${errors[name] ? 'is-invalid' : ''} ${disabled ? 'bg-light' : ''}`}
        min={min}
        disabled={disabled}
        placeholder={placeholder}
      />
      {errors[name] && <div className="invalid-feedback">{errors[name]}</div>}
    </div>
));

// Componente para selects (Bootstrap style)
const Select = React.memo(({ name, label, value, onChange, options, errors, formatFactor = (f) => '' }) => (
    <div className="mb-3">
      <label htmlFor={name} className="form-label">{label}</label>
      <select
        name={name} 
        id={name} 
        value={value} 
        onChange={onChange} 
        className={`form-select ${errors[name] ? 'is-invalid' : ''}`}
      >
        <option value="">-- Seleccione una opción --</option>
        {options.map(opt => (
          <option key={opt} value={opt}>{opt} {formatFactor(opt)}</option>
        ))}
      </select>
      {errors[name] && <div className="invalid-feedback">{errors[name]}</div>}
    </div>
));

// Componente para mostrar mensajes de feedback (Corregido para manejar strings)
const FeedbackMessage = ({ message, type }) => {
  if (!message) return null;

  const alertClass = type === 'error' ? 'alert-danger' : 'alert-success';
  const Icon = type === 'error' ? AlertCircle : CheckCircle;

  return (
    <div className={`alert ${alertClass} d-flex align-items-center`} role="alert">
      <Icon className="me-2" size={20} />
      <div>{message}</div>
    </div>
  );
};

// Componente Indicador de Pasos
const StepIndicator = ({ currentStep, totalSteps }) => (
  <div className="d-flex justify-content-center align-items-center space-x-2 space-x-md-4 mb-5">
    {Array.from({ length: totalSteps }).map((_, index) => (
      <React.Fragment key={index}>
        <div 
          className={`d-flex align-items-center justify-content-center p-2 rounded-circle text-white fw-bold 
            ${index + 1 === currentStep ? 'bg-primary border border-4 border-info' : 
             index + 1 < currentStep ? 'bg-success' : 
             'bg-secondary'}`}
          style={{ width: '35px', height: '35px' }}
        >
          {index + 1}
        </div>
        {index < totalSteps - 1 && (
          <ChevronRight className={`mx-1 ${index + 1 < currentStep ? 'text-success' : 'text-secondary'}`} size={16} />
        )}
      </React.Fragment>
    ))}
  </div>
);

// --- MODAL DE DETALLES DE COTIZACIÓN ---

const DetailModal = ({ cotizacion, show, onClose, contractCotizacion, cancelCotizacion, isLoading }) => {
  if (!show || !cotizacion) return null;
  
  const { formData, costoAnualTotal, coberturasDetalle, factorRiesgo, valorInmueble, primaBase, status, id } = cotizacion;
  
  return (
    <div className={`modal d-block ${show ? 'show' : ''}`} tabIndex="-1" style={{ backgroundColor: 'rgba(0, 0, 0, 0.5)' }}>
      <div className="modal-dialog modal-dialog-centered modal-lg">
        <div className="modal-content">
          <div className="modal-header bg-primary text-white">
            <h5 className="modal-title d-flex align-items-center">
              <Info size={24} className="me-2" />
              Detalles de la Póliza ({status})
            </h5>
            <button type="button" className="btn-close btn-close-white" onClick={onClose} aria-label="Cerrar"></button>
          </div>
          <div className="modal-body p-4">
            
            {/* Sección de Cliente y Propiedad */}
            <h6 className="fw-bold text-primary border-bottom pb-2 mb-3">Datos Generales</h6>
            <div className="row g-2 mb-4">
                <div className="col-md-6"><strong>Cliente:</strong> {formData?.nombre || 'N/A'} (Edad: {formData?.edad || 'N/A'})</div>
                <div className="col-md-6"><strong>Dirección:</strong> {formData?.direccion}, {formData?.ciudad} ({formData?.provincia})</div>
                <div className="col-md-6"><strong>Propiedad:</strong> {formData?.tipoPropiedad || 'N/A'} ({formData?.m2 || 'N/A'} m²)</div>
                <div className="col-md-6"><strong>Antigüedad:</strong> {formData?.antiguedad || 'N/A'}</div>
            </div>

            {/* Sección de Costos */}
            <h6 className="fw-bold text-primary border-bottom pb-2 mb-3">Resumen Financiero</h6>
            <div className="alert alert-info py-2 d-flex justify-content-between align-items-center">
                <span className="fw-bold fs-5">COSTO ANUAL TOTAL:</span>
                <span className="fw-bolder fs-3 text-dark">{formatCurrency(costoAnualTotal)}</span>
            </div>
            
            <div className="row g-2 mb-4">
                <div className="col-md-6"><strong>Valor Estimado Inmueble:</strong> {formatCurrency(valorInmueble)}</div>
                <div className="col-md-6"><strong>Prima Base Estructural:</strong> {formatCurrency(primaBase)}</div>
                <div className="col-12"><strong>Factor de Riesgo Aplicado:</strong> {factorRiesgo.toFixed(2)}x</div>
            </div>

            {/* Sección de Coberturas */}
            <h6 className="fw-bold text-primary border-bottom pb-2 mb-3">Coberturas Incluidas</h6>
            <ul className="list-group list-group-flush">
              {coberturasDetalle?.map((c, index) => (
                <li key={index} className="list-group-item d-flex justify-content-between">
                  <div>{c.nombre} <small className="text-secondary">({c.descripcion})</small></div>
                  <span className="fw-medium">{formatCurrency(c.costo)}</span>
                </li>
              )) || <li className="list-group-item text-muted">No se incluyeron coberturas adicionales.</li>}
            </ul>

          </div>
          <div className="modal-footer d-flex justify-content-between">
            <span className="text-muted small">ID de Cotización: {id}</span>
            <div className="d-flex gap-2">
                {status === 'Guardada' && (
                    <button 
                        onClick={() => contractCotizacion(id)}
                        className="btn btn-success fw-semibold"
                        disabled={isLoading}
                    >
                        Contratar Ahora
                    </button>
                )}
                {status === 'Contratada' && (
                    <button 
                        onClick={() => cancelCotizacion(id)}
                        className="btn btn-danger fw-semibold d-flex align-items-center"
                        disabled={isLoading}
                    >
                        <XCircle size={16} className="me-1"/> Cancelar Póliza
                    </button>
                )}
                <button type="button" className="btn btn-secondary" onClick={onClose}>Cerrar</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};


// --- COMPONENTE PRINCIPAL APP ---

const App = () => {
  const [formData, setFormData] = useState(ESTADO_INICIAL);
  const [view, setView] = useState('FORM');
  const [step, setStep] = useState(1);
  const [errors, setErrors] = useState({});
  const [cotizacionFinal, setCotizacionFinal] = useState(null);
  const [isLoading, setIsLoading] = useState(true); // Se usa para la UI de guardado
  const [cotizacionesGuardadas, setCotizacionesGuardadas] = useState([]);
  const [message, setMessage] = useState(null); // String del mensaje
  const [messageType, setMessageType] = useState('success'); // Tipo de mensaje: 'success' o 'error'
  
  // Estado para el modal de detalles
  const [showModal, setShowModal] = useState(false);
  const [selectedCotizacion, setSelectedCotizacion] = useState(null);
  
  // Carga inicial y configuración de Bootstrap
  useEffect(() => {
    // Carga Bootstrap
    if (!document.querySelector('link[href*="bootstrap"]')) {
      const link = document.createElement('link');
      link.href = 'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css';
      link.rel = 'stylesheet';
      document.head.appendChild(link);
    }
    
    // Carga inicial de datos desde localStorage
    const initialData = loadFromLocalStorage();
    setCotizacionesGuardadas(initialData.sort((a, b) => b.fechaGuardado.getTime() - a.fechaGuardado.getTime()));
    setIsLoading(false); // La carga local es instantánea
  }, []);

  // --- LÓGICA DE PERSISTENCIA (localStorage) ---
  
  const updateCotizacionesList = useCallback((updatedList) => {
      // Ordena por fecha de guardado descendente
      const sortedList = updatedList.sort((a, b) => b.fechaGuardado.getTime() - a.fechaGuardado.getTime());
      setCotizacionesGuardadas(sortedList);
      saveToLocalStorage(sortedList);
  }, []);

  const saveCotizacion = useCallback((cotizacionData, initialStatus = 'Guardada') => {
    setIsLoading(true);
    setMessage(null);
    setMessageType('success');
    
    // Genera un ID simple basado en el timestamp actual
    const newCotizacion = {
        id: Date.now().toString(),
        ...cotizacionData,
        status: initialStatus,
        fechaGuardado: new Date(),
        formData: formData,
    };

    const updatedList = [...cotizacionesGuardadas, newCotizacion];
    updateCotizacionesList(updatedList);

    const msg = initialStatus === 'Contratada' 
      ? `¡Póliza contratada con éxito! ID: ${newCotizacion.id}`
      : `Cotización guardada exitosamente con ID: ${newCotizacion.id}`;
      
    setMessage(msg);
    setMessageType('success');
    handleReset();
    setIsLoading(false);
  }, [formData, cotizacionesGuardadas, updateCotizacionesList]);
  
  const updateCotizacionStatus = useCallback((id, newStatus) => {
    setIsLoading(true);
    setMessage(null);
    setMessageType('success');
    setShowModal(false); 

    const updatedList = cotizacionesGuardadas.map(c => {
        if (c.id === id) {
            const update = { status: newStatus };
            if (newStatus === 'Contratada') update.fechaContratacion = new Date();
            if (newStatus === 'Cancelada') update.fechaCancelacion = new Date();
            return { ...c, ...update };
        }
        return c;
    });

    updateCotizacionesList(updatedList);

    const successMessage = newStatus === 'Contratada' 
        ? `¡Póliza contratada con éxito! ID: ${id}`
        : `Suscripción del seguro cancelada correctamente. ID: ${id}`;
        
    setMessage(successMessage);
    setMessageType('success');
    setIsLoading(false);
  }, [cotizacionesGuardadas, updateCotizacionesList]);

  const contractCotizacion = (id) => updateCotizacionStatus(id, 'Contratada');
  const cancelCotizacion = (id) => updateCotizacionStatus(id, 'Cancelada');
  
  // --- LÓGICA DE MODAL ---
  const handleViewDetails = useCallback((cotizacion) => {
    setSelectedCotizacion(cotizacion);
    setShowModal(true);
  }, []);

  const handleCloseModal = useCallback(() => {
    setShowModal(false);
    setSelectedCotizacion(null);
  }, []);

  // --- LÓGICA DE FORMULARIO Y CÁLCULO ---

  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => {
      if (type === 'checkbox') {
        const current = prev.coberturasSeleccionadas;
        const updated = checked 
          ? [...current, name] 
          : current.filter(id => id !== name);
        return { ...prev, coberturasSeleccionadas: updated };
      }
      return { ...prev, [name]: value };
    });
    setErrors(prev => ({ ...prev, [name]: undefined }));
  }, []); 

  const validateStep = useCallback(() => {
    let newErrors = {};

    if (step === 1) { 
      if (!formData.nombre.trim()) newErrors.nombre = 'El nombre es requerido.';
      if (!formData.edad || isNaN(formData.edad) || formData.edad < 18) newErrors.edad = 'Edad inválida. Debe ser mayor de 18.';
      if (!formData.m2 || isNaN(formData.m2) || formData.m2 <= 0) newErrors.m2 = 'Los metros cuadrados son requeridos y deben ser un número positivo.';
      if (!formData.provincia) newErrors.provincia = 'La provincia es requerida.';
      if (!formData.ciudad) newErrors.ciudad = 'La ciudad es requerida.';
      if (!formData.direccion) newErrors.direccion = 'La dirección es requerida.';
    } else if (step === 2) { 
      if (!formData.ubicacion) newErrors.ubicacion = 'Debe seleccionar la zona de ubicación.';
      if (!formData.tipoPropiedad) newErrors.tipoPropiedad = 'Debe seleccionar el tipo de propiedad.';
      if (!formData.antiguedad) newErrors.antiguedad = 'Debe seleccionar la antigüedad.';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  }, [formData, step]);

  const calcularCotizacion = useCallback(() => {
    const { m2, ubicacion, tipoPropiedad, antiguedad, coberturasSeleccionadas } = formData;
    const valorInmueble = parseFloat(m2) * VALOR_BASE_M2;
    const factorUbicacion = FACTORES.ubicacion[ubicacion] || 1;
    const factorTipo = FACTORES.tipoPropiedad[tipoPropiedad] || 1;
    const factorAntiguedad = FACTORES.antiguedad[antiguedad] || 1;
    const factorRiesgoTotal = factorUbicacion * factorTipo * factorAntiguedad;
    // Se usa 0.005 como porcentaje de la prima base sobre el valor del inmueble
    const primaBase = valorInmueble * 0.005 * factorRiesgoTotal; 

    let costoCoberturasAdicionales = 0;
    const coberturasDetalle = COBERTURAS_BASE
      .filter(c => coberturasSeleccionadas.includes(c.id))
      .map(c => {
        const costoCobertura = valorInmueble * c.costoFactor * factorRiesgoTotal;
        costoCoberturasAdicionales += costoCobertura;
        return { ...c, costo: costoCobertura };
      });

    const costoAnualTotal = primaBase + costoCoberturasAdicionales;

    const opciones = [
      { nombre: 'Plan Mensual', costo: costoAnualTotal / 12, ahorro: 0, esDestacado: false },
      // 3% de descuento por pago semestral (se calcula sobre la mitad del costo anual)
      { nombre: 'Plan Semestral', costo: (costoAnualTotal * 0.97) / 2, ahorro: costoAnualTotal * 0.03, esDestacado: false }, 
      { nombre: 'Plan Anual (10% OFF)', costo: costoAnualTotal * 0.9, ahorro: costoAnualTotal * 0.1, esDestacado: true }, 
    ];
    
    setCotizacionFinal({
      valorInmueble, factorRiesgo: factorRiesgoTotal, primaBase, coberturasDetalle, costoAnualTotal, opcionesPago: opciones,
      datosCliente: { nombre: formData.nombre, edad: formData.edad },
    });
  }, [formData]);

  const handleNext = () => {
    if (validateStep()) {
      setMessage(null); // Limpiar mensaje al avanzar
      if (step === 3) {
        calcularCotizacion();
        setStep(4);
      } else {
        setStep(prev => prev + 1);
      }
    } else {
        setMessage('Por favor, complete todos los campos requeridos y corrija los errores para avanzar.');
        setMessageType('error');
    }
  };
  
  // FUNCIÓN AGREGADA para manejar el botón "Anterior"
  const handleBack = () => {
    setMessage(null);
    setErrors({});
    if (step > 1) {
      setStep(prev => prev - 1);
    }
  };

  const handleReset = () => {
    setFormData(ESTADO_INICIAL);
    setCotizacionFinal(null);
    setErrors({});
    setStep(1);
    setMessage(null);
    setMessageType('success');
    setView('FORM');
    handleCloseModal(); 
  };


  const ubicacionOptions = useMemo(() => Object.keys(FACTORES.ubicacion), []);
  const tipoPropiedadOptions = useMemo(() => Object.keys(FACTORES.tipoPropiedad), []);
  const antiguedadOptions = useMemo(() => Object.keys(FACTORES.antiguedad), []);
  
  // Función para formatear el factor de riesgo para el Select
  const formatFactor = useCallback((option) => {
    if (FACTORES.ubicacion[option]) return `(${formatPercent(FACTORES.ubicacion[option])})`;
    if (FACTORES.tipoPropiedad[option]) return `(${formatPercent(FACTORES.tipoPropiedad[option])})`;
    if (FACTORES.antiguedad[option]) return `(${formatPercent(FACTORES.antiguedad[option])})`;
    return '';
  }, []);


  // --- RENDERIZADO DE PASOS ---

  const renderStep1 = () => (
    <>
      <h2 className="fs-4 fw-semibold text-dark mb-4 d-flex align-items-center">
        <User className="me-2 text-primary" size={24} /> 1. Datos del Solicitante y Propiedad
      </h2>
      <div className="row">
        {/* Datos Personales */}
        <div className="col-md-6">
          <Input name="nombre" label="Nombre Completo" value={formData.nombre} onChange={handleChange} errors={errors} placeholder="Ej: Juan Pérez" />
        </div>
        <div className="col-md-6">
          <Input name="edad" label="Edad" type="number" value={formData.edad} onChange={handleChange} errors={errors} min="18" placeholder="Ej: 35" />
        </div>
        <div className="col-12">
          <Input name="m2" label="Metros Cuadrados (m²)" type="number" value={formData.m2} onChange={handleChange} errors={errors} min="1" placeholder="Ej: 150" />
        </div>
      </div>
        
      {/* Datos de Ubicación Detallada */}
      <div className="p-4 rounded border bg-light mt-3">
        <h3 className="fs-5 fw-medium text-secondary mb-3">Ubicación a Asegurar</h3>
        <Input name="pais" label="País" value={formData.pais} onChange={handleChange} errors={errors} disabled={true} />
        <div className="row">
          <div className="col-md-6">
            <Input name="provincia" label="Provincia" value={formData.provincia} onChange={handleChange} errors={errors} placeholder="Ej: Buenos Aires" />
          </div>
          <div className="col-md-6">
            <Input name="ciudad" label="Ciudad" value={formData.ciudad} onChange={handleChange} errors={errors} placeholder="Ej: Balcarce" />
          </div>
        </div>
        <Input name="direccion" label="Dirección Completa" value={formData.direccion} onChange={handleChange} errors={errors} placeholder="Ej: Calle Falsa 123" />
      </div>
    </>
  );

  const renderStep2 = () => (
    <>
      <h2 className="fs-4 fw-semibold text-dark mb-4 d-flex align-items-center">
        <Home className="me-2 text-primary" size={24} /> 2. Factores de Riesgo de la Propiedad
      </h2>
      
      <Select 
        name="ubicacion" 
        label="Zona de Riesgo (Factor de Cotización)"
        value={formData.ubicacion} 
        onChange={handleChange} 
        options={ubicacionOptions}
        errors={errors}
        formatFactor={formatFactor}
      />
      
      <Select 
        name="tipoPropiedad" 
        label="Tipo de Propiedad"
        value={formData.tipoPropiedad} 
        onChange={handleChange} 
        options={tipoPropiedadOptions}
        errors={errors}
        formatFactor={formatFactor}
      />
      
      <Select 
        name="antiguedad" 
        label="Antigüedad"
        value={formData.antiguedad} 
        onChange={handleChange} 
        options={antiguedadOptions}
        errors={errors}
        formatFactor={formatFactor}
      />
    </>
  );

  const renderStep3 = () => (
    <>
      <h2 className="fs-4 fw-semibold text-dark mb-4 d-flex align-items-center">
        <DollarSign className="me-2 text-primary" size={24} /> 3. Opciones de Cobertura
      </h2>
      <p className="text-secondary mb-4">Seleccione las coberturas adicionales que desea incluir en su póliza. El seguro básico incluye cobertura estructural mínima.</p>
      
      <div className="space-y-3">
        {COBERTURAS_BASE.map(cobertura => (
          <div key={cobertura.id} className="card shadow-sm border-0 mb-3">
            <div className="card-body p-4">
              <div className="form-check d-flex align-items-start">
                <input 
                  type="checkbox" 
                  name={cobertura.id} 
                  checked={formData.coberturasSeleccionadas.includes(cobertura.id)}
                  onChange={handleChange}
                  className="form-check-input flex-shrink-0 mt-1 me-3"
                  id={`check-${cobertura.id}`}
                />
                <label className="form-check-label w-100" htmlFor={`check-${cobertura.id}`}>
                  <p className="mb-0 fw-medium text-dark">{cobertura.nombre}</p>
                  <small className="text-secondary">{cobertura.descripcion}</small>
                </label>
              </div>
            </div>
          </div>
        ))}
      </div>
    </>
  );

  const renderStep4 = () => (
    <>
      <h2 className="fs-3 fw-bold text-primary mb-4 text-center d-flex align-items-center justify-content-center">
        <Calculator className="me-3" size={32} /> Resultado de Cotización
      </h2>

      {cotizacionFinal ? (
        <div className="space-y-4">
          
          {/* Resumen del Cliente y Propiedad */}
          <div className="card shadow border-0 bg-primary-subtle">
            <div className="card-body p-4">
              <h3 className="fs-5 fw-semibold text-primary border-bottom pb-2 mb-3">Resumen de su Póliza</h3>
              <p className="text-dark mb-2"><strong>Dirección:</strong> {formData.direccion}, {formData.ciudad} ({formData.provincia}, {formData.pais})</p>
              <div className="row">
                  <div className="col-md-6"><p className="mb-1"><strong>Metros Cuadrados:</strong> {formData.m2} m²</p></div>
                  <div className="col-md-6"><p className="mb-1"><strong>Tipo de Propiedad:</strong> {formData.tipoPropiedad}</p></div>
                  <div className="col-md-6"><p className="mb-1"><strong>Valor Estimado:</strong> <span className="fw-bold text-success">{formatCurrency(cotizacionFinal.valorInmueble)}</span></p></div>
                  <div className="col-md-6"><p className="mb-1"><strong>Factor de Riesgo:</strong> {(cotizacionFinal.factorRiesgo).toFixed(2)}x</p></div>
              </div>
            </div>
          </div>

          {/* Detalles de Costos */}
          <div className="card shadow border-0">
            <div className="card-body p-4">
              <h3 className="fs-5 fw-semibold text-dark border-bottom pb-2 mb-3">Desglose Anual</h3>
              <ul className="list-group list-group-flush mb-3">
                <li className="list-group-item d-flex justify-content-between align-items-center bg-light">
                  <span>Prima Base Estructural:</span>
                  <span className="fw-medium">{formatCurrency(cotizacionFinal.primaBase)}</span>
                </li>
                {cotizacionFinal.coberturasDetalle.map(c => (
                  <li key={c.id} className="list-group-item d-flex justify-content-between align-items-center border-start border-4 border-info">
                    <span>+ {c.nombre}:</span>
                    <span className="fw-medium">{formatCurrency(c.costo)}</span>
                  </li>
                ))}
              </ul>
              <div className="pt-3 border-top border-primary-subtle d-flex justify-content-between align-items-center">
                <span className="fs-5 fw-bold text-primary">COSTO ANUAL TOTAL:</span>
                <span className="fs-3 fw-bolder text-dark">{formatCurrency(cotizacionFinal.costoAnualTotal)}</span>
              </div>
            </div>
          </div>

          {/* Opciones de Pago */}
          <div className="card shadow border-0">
            <div className="card-body p-4">
              <h3 className="fs-5 fw-semibold text-dark border-bottom pb-2 mb-3">Planes de Pago</h3>
              <div className="row row-cols-1 row-cols-md-3 g-3">
                {cotizacionFinal.opcionesPago.map((opcion, index) => (
                  <div key={index} className="col">
                    <div className={`card h-100 ${opcion.esDestacado ? 'border-success border-3' : 'border-light'}`}>
                      <div className={`card-header text-center fw-bold ${opcion.esDestacado ? 'bg-success text-white' : 'bg-light'}`}>
                        {opcion.nombre}
                      </div>
                      <div className="card-body text-center">
                        <p className="fs-4 fw-bolder text-primary mb-1">{formatCurrency(opcion.costo)}</p>
                        {/* Se ajusta el cálculo para que el Plan Mensual muestre la cuota si es el plan seleccionado */}
                        {!opcion.esDestacado && opcion.nombre !== 'Plan Mensual' && (
                             <p className="text-secondary mb-0">Cuota: {formatCurrency(opcion.costo / (opcion.nombre === 'Plan Semestral' ? 6 : 1))}</p>
                        )}
                        {opcion.ahorro > 0 && (
                          <p className="badge text-bg-success mt-2">¡Ahorras {formatCurrency(opcion.ahorro)}!</p>
                        )}
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
          
          {/* Acciones de Cotización */}
          <div className="d-flex justify-content-end gap-3 pt-3 border-top">
            <button 
              onClick={() => saveCotizacion(cotizacionFinal)} 
              disabled={isLoading}
              className="btn btn-warning fw-semibold d-flex align-items-center"
            >
              <Save size={20} className="me-2" />
              Guardar Cotización
            </button>
            <button 
              onClick={() => {
                saveCotizacion(cotizacionFinal, 'Contratada'); 
                setView('LIST');
              }}
              disabled={isLoading}
              className="btn btn-success fw-semibold d-flex align-items-center"
            >
              <CheckCircle size={20} className="me-2" />
              Contratar Ahora
            </button>
          </div>
        </div>
      ) : (
        <FeedbackMessage message="Error al calcular la cotización. Por favor, revise los datos ingresados." type="error" />
      )}
    </>
  );

  // Nueva vista: Mis Pólizas y Cotizaciones Guardadas
  const renderListView = () => (
    <div className="pb-4">
      <h2 className="fs-3 fw-bold text-primary mb-4 d-flex align-items-center">
        <List className="me-3" size={32} /> Mis Pólizas y Cotizaciones Guardadas
      </h2>
      
      {isLoading && <div className="text-center text-primary py-5">Cargando datos...</div>}
      
      {cotizacionesGuardadas.length === 0 && !isLoading ? (
        <div className="text-center p-5 bg-light rounded border border-dashed">
          <p className="lead text-secondary">Aún no tienes cotizaciones guardadas ni pólizas activas.</p>
          <button onClick={handleReset} className="btn btn-link fw-medium">
            Comenzar nueva cotización
          </button>
        </div>
      ) : (
        <div className="space-y-3">
          {cotizacionesGuardadas.map(cotizacion => (
            <div key={cotizacion.id} className={`card shadow-sm border-start border-5 
              ${cotizacion.status === 'Contratada' ? 'border-success bg-light' :
               cotizacion.status === 'Cancelada' ? 'border-danger bg-light' :
               'border-primary bg-white'}`}>
              
              <div className="card-body">
                <div className="d-flex justify-content-between align-items-start mb-2">
                  <h3 className="fs-5 fw-semibold text-dark">
                    {cotizacion.status === 'Contratada' ? 'Póliza Activa' : cotizacion.status === 'Cancelada' ? 'Póliza Cancelada' : 'Cotización Guardada'}
                  </h3>
                  <span className={`badge text-uppercase fw-bold 
                    ${cotizacion.status === 'Contratada' ? 'bg-success' :
                     cotizacion.status === 'Cancelada' ? 'bg-danger' :
                     'bg-primary'}`}>
                    {cotizacion.status}
                  </span>
                </div>
                
                <p className="text-secondary mb-2">
                  <span className="fw-semibold">Dirección:</span> {cotizacion.formData?.direccion || 'N/A'} 
                </p>
                <p className="text-secondary mb-3">
                  <span className="fw-semibold">Guardado:</span> {formatDate(cotizacion.fechaGuardado)}
                </p>
                
                <div className="fs-4 fw-bolder text-primary mb-3">
                  Costo Anual: {formatCurrency(cotizacion.costoAnualTotal)}
                </div>

                {/* Botones de Acción */}
                <div className="d-flex gap-2 pt-2 border-top">
                  {/* BOTÓN DE DETALLES */}
                  <button 
                      onClick={() => handleViewDetails(cotizacion)}
                      className="btn btn-sm btn-outline-info fw-semibold d-flex align-items-center"
                    >
                      <Info size={16} className="me-1"/> Ver Detalles
                    </button>
                  {cotizacion.status === 'Guardada' && (
                    <button 
                      onClick={() => contractCotizacion(cotizacion.id)}
                      className="btn btn-sm btn-success fw-semibold"
                      disabled={isLoading}
                    >
                      Contratar Póliza
                    </button>
                  )}
                  {cotizacion.status === 'Contratada' && (
                    <button 
                      onClick={() => cancelCotizacion(cotizacion.id)}
                      className="btn btn-sm btn-danger fw-semibold d-flex align-items-center"
                      disabled={isLoading}
                    >
                      <XCircle size={16} className="me-1"/> Cancelar Suscripción
                    </button>
                  )}
                </div>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );


  // --- RENDERIZADO PRINCIPAL ---

  return (
    <div className="bg-light min-vh-100 py-5">
      <header className="text-center mb-5">
        <h1 className="display-5 fw-bolder text-primary mb-2">
          Cotizador de Seguros para el Hogar
        </h1>
        <p className="lead text-secondary">Desarrollo con React y Bootstrap 5 (Guardado en localStorage)</p>
      </header>
      
      {/* Mensajes de feedback */}
      <div className="container" style={{ maxWidth: '900px' }}>
        <FeedbackMessage message={message} type={messageType} />
      </div>

      <div className="container bg-white p-4 p-md-5 rounded-3 shadow-lg" style={{ maxWidth: '900px' }}>
        
        {/* Navegación Principal */}
        <div className="d-flex justify-content-end gap-3 mb-4">
            <button 
                onClick={() => setView('FORM')}
                className={`btn fw-semibold ${view === 'FORM' ? 'btn-primary' : 'btn-outline-secondary'}`}
            >
                <Calculator className="me-1" size={18}/> Nueva Cotización
            </button>
            <button 
                onClick={() => setView('LIST')}
                className={`btn fw-semibold ${view === 'LIST' ? 'btn-primary' : 'btn-outline-secondary'}`}
            >
                <List className="me-1" size={18}/> Mis Pólizas <span className="badge text-bg-secondary ms-1">{cotizacionesGuardadas.length}</span>
            </button>
        </div>

        {/* Contenido según la vista */}
        {view === 'FORM' ? (
          <>
            <StepIndicator currentStep={step} totalSteps={4} />
            <div className="min-vh-50 transition-all duration-500 ease-in-out">
              {step === 1 && renderStep1()}
              {step === 2 && renderStep2()}
              {step === 3 && renderStep3()}
              {step === 4 && renderStep4()}
            </div>
            
            {/* Controles de Navegación del Formulario */}
            <div className="mt-5 pt-4 border-top d-flex justify-content-between">
              
              {/* CORRECCIÓN: Se utiliza la función handleBack definida */}
              {step > 1 && step < 4 && (
                <button 
                  onClick={handleBack} 
                  className="btn btn-secondary fw-semibold"
                >
                  Anterior
                </button>
              )}

              {step === 4 && (
                <button 
                  onClick={handleReset} 
                  className="btn btn-outline-danger fw-semibold d-flex align-items-center"
                >
                  <RefreshCw size={20} className="me-2" />
                  Nueva Cotización
                </button>
              )}

              <div className="flex-grow-1 d-flex justify-content-end">
                {step < 4 && (
                  <button 
                    onClick={handleNext} 
                    className="btn btn-primary fw-semibold"
                  >
                    {step === 3 ? 'Calcular Cotización' : 'Siguiente'}
                  </button>
                )}
              </div>
            </div>
          </>
        ) : (
          renderListView()
        )}
      </div>
      
      {/* El modal de detalles se renderiza aquí */}
      <DetailModal 
        cotizacion={selectedCotizacion}
        show={showModal}
        onClose={handleCloseModal}
        formatCurrency={formatCurrency}
        formatDate={formatDate}
        contractCotizacion={contractCotizacion}
        cancelCotizacion={cancelCotizacion}
        isLoading={isLoading}
      />
      
      {/* Overlay del modal */}
      {showModal && <div className="modal-backdrop fade show"></div>}
      
    </div>
  );
};

export default App;