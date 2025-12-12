> 🇬🇧 Read this in English: [README.md](README.md)

# Subí que te llevo – Plan y resultados de testing
Evaluación integral de un sistema de gestión de viajes (TP de Taller de Programación I) con pruebas unitarias, de integración, persistencia, mocks y GUI. Se verifica la lógica de negocio, la persistencia binaria y los flujos de ventana para asegurar que choferes, vehículos, pedidos y viajes se gestionen según el contrato.

[Java 8](#como-ejecutar) · [Maven](#como-ejecutar) · [JUnit 4](#tipos-de-pruebas-y-suites-con-ejemplos-del-repo) · [Mockito](#mocks-mockito-y-moc) · [Robot GUI](#pruebas-de-gui-robot)

## Índice
- [Contexto del TP](#contexto-del-tp)
- [Sistema bajo prueba](#sistema-bajo-prueba)
- [Estrategia de testing](#estrategia-de-testing)
- [Tipos de pruebas y suites (con ejemplos)](#tipos-de-pruebas-y-suites-con-ejemplos-del-repo)
- [Escenarios y datos de prueba clave](#escenarios-y-datos-de-prueba-clave)
- [Mapa breve de cobertura](#mapa-breve-de-cobertura)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Cómo ejecutar](#como-ejecutar)
- [Resultados del testing](#resultados-del-testing)
- [Conclusión del trabajo de testing](#conclusion-del-trabajo-de-testing)
- [Créditos y licencia](#creditos-y-licencia)

## Contexto funcional de la app
**Sistema de gestión de viajes.** Empresa de transporte con flota variada (Auto/Combi/Moto), choferes permanentes/temporarios y clientes registrados; los recursos crecen con altas sucesivas.

**Usuarios**: 1 administrador y varios clientes.

**Administrador**
- Alta de choferes y vehículos.
- Listados: clientes, choferes, vehículos, viajes; puntaje/sueldo de chofer; total de sueldos.
- Crear viaje asignando chofer y vehículo disponibles a un pedido pendiente.

**Cliente**
- Registro de usuario (sin duplicar nombre).
- Solicitud de viaje: aceptación o rechazo por falta de vehículo pertinente (capacidad, baúl, mascota).
- Pago y calificación del chofer; historial de viajes propios.

**Contratos clave**
- Sin duplicados de usuarios/vehículos/choferes.
- Máximo 10 pasajeros por pedido.
- Selección de vehículo por prioridad y aptitud (tipo/capacidad/baúl/mascota).
- Cálculo de costo por zona, equipaje, mascotas y km; sueldos según tipo de chofer (bruto/neto).

## Contexto del TP
- **Enunciado** (`EnunciadoTP.docx`): un solo administrador (“admin”/“admin”) y múltiples clientes; alta/listados de choferes, vehículos y viajes; creación/aceptación de pedidos; cálculo de salarios y calificación de choferes; sin bajas.
- **Pautas** (`PautasTP.docx`): proyecto Maven con tests de caja negra/blanca (cubrir líneas faltantes), integración orientada a objetos, persistencia (lectura/escritura con archivo existente o inexistente), GUI, excepciones y conclusiones.
- **SRS** (`SRS.docx`): RF/RNF de usuarios, vehículos, pedidos y viajes; autenticación; cálculo de costos; sueldos; persistencia binaria; sin duplicados; máximo 10 pasajeros.
- **Comportamiento de vistas** (`Comportamiento_Vistas.docx`): reglas de habilitado/deshabilitado y mensajes en Login, Registro, Cliente y Administrador (las pruebas GUI siguen estos criterios).
- **Planillas de prueba**: `Modelo de Datos - Tabla de Particiones y Bateria de Prueba.xlsx` y `_Persistencia - Tabla de Particiones y Bateria de Prueba.xlsx` definen clases de equivalencia y valores límite usados en los tests.

## Sistema bajo prueba
- **Actores**: 1 administrador y varios clientes.
- **Admin**: alta de choferes/vehículos, listados (clientes/choferes/vehículos/viajes), total de sueldos, viajes por chofer, creación de viaje asignando chofer y vehículo disponibles.
- **Cliente**: registro, solicitud de viaje (aceptación o rechazo por falta de vehículo), pago y calificación, historial.
- **Módulos**: `modeloDatos` (Administrador, Cliente, Chofer Permanente/Temporario, Vehiculo Moto/Auto/Combi, Pedido, Viaje), `modeloNegocio.Empresa` (singleton), `controlador.Controlador`, `vista.Ventana` + `IOptionPane`.

## Estrategia de testing
- **Diseño**: clases de equivalencia y valores frontera de las planillas para cada constructor/método (patente no nula, plazas en rango, zonas válidas, archivo de persistencia existente/inexistente/nulo, etc.).
- **Escenarios reutilizables** (negocio): `EscenarioBase` (vacío) y `Escenario1–5` para clientes sin viajes, con pedidos pendientes, con viajes iniciados y con viajes terminados. Persistencia: archivo inexistente, vacío y con datos.
- **Alcance**: se testean métodos públicos según contrato; se omiten getters/setters generados. Se validan mensajes y excepciones (`Mensajes`, `Constantes`). GUI/persistencia se aíslan para no depender de infraestructura externa.

## Tipos de pruebas y suites (con ejemplos del repo)
- **Unitarias – modelo de datos** (`TP/src/test/java/modelo/dato`)
  - `AutoTest_ConMascota` y `_SinMascota`: puntaje 80/60 según baúl; null si excede plazas o no admite mascota.
  - `MotoTest`: 1000 puntos solo si es 1 pax sin baúl/mascota; null en casos inválidos.
  - `CombiTest_ConMascota/_SinMascota`: 10 * pax con/sin 100 extra por baúl; null si supera plazas.
  - `ViajeTest_Esc1/2/3`: costo por zona + baúl + mascota; finalización y calificación.
  - `ChoferPermanenteTest/ChoferTemporarioTest`: sueldo bruto según antigüedad e hijos; sueldo neto 86%.
- **Unitarias de negocio** (`TP/src/test/java/modelo/negocio`)
  - `EmpresaTestEscenarioVacio`: altas válidas y duplicados (`UsuarioYaExiste`, `VehiculoRepetido`), login de usuario inexistente, salario total sin choferes.
  - `EmpresaTestEscenario1`: login/logout correcto y password errónea (mensaje y datos en excepción).
  - `EmpresaTestEscenario2–5`: `agregarPedido` (vehículo no disponible, cliente con pedido/viaje), `vehiculosOrdenadosPorPedido` (orden y filtrado), `crearViaje` (chofer/vehículo no disponible o no válido), `calificacionDeChofer`, `getTotalSalarios`.
- **Integración (Controlador + Vista mockeada)** (`TP/src/test/java/controlador`)
  - `ControladorTest1` (empresa vacía) y `ControladorTest2` (listas pobladas) usando Mockito sobre `IVista` y `OptionPane` falso.
  - Flujos: login correcto/incorrecto, registro (passwords no coinciden, usuario repetido), alta de chofer/vehículo, pedidos (sin vehículo, cliente con pedido/viaje), creación de viaje (pedido inexistente, chofer/vehículo no disponible, vehículo no válido), calificar/pagar con y sin viaje.
- **Persistencia** (`TP/src/test/java/persistencia`)
  - `PersistenciaBinTest_Esc1/2/3`: abrir/cerrar input-output, escribir/leer choferes, vehículos, clientes, pedidos, viajes iniciados/terminados y usuario logueado. Casos: archivo inexistente, nulo, sin permisos, objeto no serializable.
  - `EmpresaDTOTest`: conversión Empresa ↔ EmpresaDTO y consistencia de colecciones.
- **Mocks (Mockito) y “MOC”**
  - Mockito para stub de `IVista`; `OptionPane`/`FalseOptionPanel` capturan mensajes sin diálogos reales.
  - “MOC” en la materia refiere al uso de dobles/mocks para aislar dependencias (aquí se implementa con Mockito y OptionPane falso).
- **Pruebas de GUI (Robot)** (`TP/src/test/java/Vista/testGUI`)
  - Suite `AllTestGui`: habilitado/deshabilitado en Login/Registro, alta duplicada de chofer/vehículo (`Mensajes.CHOFER_YA_REGISTRADO`, `Mensajes.VEHICULO_YA_REGISTRADO`), creación de viaje desde panel Admin y flujo de pedido/pago en panel Cliente según `Comportamiento_Vistas.docx`.

## Escenarios y datos de prueba clave
- `EscenarioBase`: empresa vacía (se usa como base).
- `Escenario1`: clientes sin pedidos/viajes.
- `Escenario2`: clientes + choferes + vehículos sin pedidos (orden de vehículos, altas).
- `Escenario3`: pedidos pendientes múltiples (excepciones de pedidos y disponibilidad).
- `Escenario4`: viajes en curso (chofer/vehículo ocupados).
- `Escenario5`: viajes finalizados y calificados (promedios y sueldos).
- Persistencia: archivo `empresa.bin` inexistente, vacío o con datos; apertura/cierre nulo; escritura/lectura de objetos serializables.

## Mapa breve de cobertura
| Requisito funcional | Tests relevantes |
| --- | --- |
| Alta de chofer | `controlador.ControladorTest1#testNuevoChofer`, `controlador.ControladorTest2#testNuevoChofer`, GUI `TestGuiAdmEsc2` |
| Alta de vehículo | `ControladorTest1#testNuevoVehiculo`, `ControladorTest2#testNuevoVehiculo`, GUI `TestGuiAdmEsc2` |
| Login/Registro | `ControladorTest1#testLogin`, `ControladorTest2#testLogin/testLogin2`, `ControladorTest1#testRegistrar/testRegistrar2`, GUI `AllTestGui` |
| Pedido sin vehículo apto | `ControladorTest2#testNuevoPedido2`, `EmpresaTestEscenario3` |
| Pedido con pedido/viaje pendiente | `ControladorTest2#testNuevoPedido3/testNuevoPedido4`, `EmpresaTestEscenario3/4/5` |
| Crear viaje (chofer/vehículo disponibles) | `ControladorTest2#testNuevoViaje6`, `EmpresaTestEscenario4/5` |
| Crear viaje con errores (pedido inexistente, chofer/vehículo no disponible/válido) | `ControladorTest2#testNuevoViaje`, `testNuevoViaje2/3/4` |
| Cálculo de costo de viaje | `ViajeTest_Esc1/2/3` |
| Salarios de choferes | `EmpresaTestEscenarioVacio#testGetTotalSalarios_*` |
| Persistencia lectura/escritura | `PersistenciaBinTest_Esc1/2/3`, `EmpresaDTOTest` |

## Estructura del repositorio
- Documentación y consigna: `EnunciadoTP.docx`, `PautasTP.docx`, `SRS.docx`, `Comportamiento_Vistas.docx`, `Informe.docx`.
- Planillas: `Modelo de Datos - Tabla de Particiones y Bateria de Prueba.xlsx`, `_Persistencia - Tabla de Particiones y Bateria de Prueba.xlsx`.
- Código y tests: `TP/src/test/java/` en `controlador`, `modelo/dato`, `modelo/negocio`, `persistencia`, `Vista/testGUI`.
- Build/dependencias: `TP/pom.xml` (JUnit 4.13.2, Mockito 3.2.4, `lib/SubiQueTeLlevo.jar` como dependencia `system`), binarios de ejemplo `TP/datos.bin`, `TP/empresa.bin`.
- Reportes: `ResultadosTesteo.pdf` y `TP/target/site/surefire-report.html` (al ejecutar `mvn test`).

## Cómo ejecutar
Requisitos: Java 8+ y Maven.

```bash
# Desde la raíz del repo
cd TP-TallerProgramacio1/TP
mvn test
```
- Suites específicas (nombres exactos):  
  - `mvn -Dtest=AllTest test` (suite raíz)  
  - `mvn -Dtest=controlador.AllTestController test`  
  - `mvn -Dtest=modelo.negocio.EmpresaTestSuite test`  
  - `mvn -Dtest=persistencia.AllTestPersistencia test`  
- Omitir GUI (entornos sin display) o usar Xvfb:  
  `mvn -Dtest=AllTest -Dsurefire.excludes=Vista/testGUI/** test`
- Si Robot falla por foco/tiempos, ejecutar local cerrando apps que roben foco y repetir.
- Reportes: `TP/target/surefire-reports/` (texto) y `TP/target/site/surefire-report.html` (HTML).

## Resultados del testing
- `Empresa.getTotalSalarios` devuelve montos menores a lo esperado (choferes permanentes/temporarios).
- `Empresa.vehiculosOrdenadosPorPedido` no ordena por puntaje y devuelve vehículos no aptos.
- `Empresa.agregarPedido` no lanza `SinVehiculoParaPedido`, `ClienteConPedidoPendiente` o `ClienteConViajePendiente` en varios escenarios.
- `Empresa.crearViaje` devuelve mensajes/excepciones incorrectas (chofer no disponible vs pedido inexistente; vehículo no disponible/válido).
- `Empresa.calificacionDeChofer` no lanza `SinViajesException` y calcula mal el promedio.
- Persistencia: conversión Empresa ↔ EmpresaDTO y lectura/escritura binaria validadas, incluyendo archivos inexistentes/vacíos.

## Conclusión del trabajo de testing
- Se cumplieron las pautas: Maven, pruebas de unidad/integración/persistencia/GUI, evidencias y escenarios derivados de tablas de partición.
- El enfoque por contrato y escenarios permitió detectar defectos en reglas de negocio y cálculos, mostrando eficacia del plan de pruebas.
- Para CI/headless, ejecutar sin GUI o con Xvfb; para reproducir issues de Robot, correr local con foco estable.

## Authors
- Criado, Facundo
- Nieto, Iván Ezequiel 
- Saladino, Juan Cruz
- San Pedro, Gianfranco