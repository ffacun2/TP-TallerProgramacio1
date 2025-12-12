> 🇪🇸 Read this in Spanish: [README_ES.md](README_ES.md)

# Subí que te llevo – Plan and testing results
Comprehensive evaluation of a ride-management system (TP for Taller de Programación I) with unit, integration, persistence, mocks, and GUI tests. It verifies business logic, binary persistence, and window flows to ensure drivers, vehicles, requests, and trips are handled according to the contract.

[Java 8](#how-to-run) · [Maven](#how-to-run) · [JUnit 4](#types-of-tests-and-suites-with-examples-from-the-repo) · [Mockito](#mocks-mockito-and-moc) · [Robot GUI](#gui-tests-robot)

## Table of Contents
- [Functional context of the app](#functional-context-of-the-app)
- [TP context](#tp-context)
- [System under test](#system-under-test)
- [Testing strategy](#testing-strategy)
- [Types of tests and suites (with examples)](#types-of-tests-and-suites-with-examples-from-the-repo)
- [Key scenarios and test data](#key-scenarios-and-test-data)
- [Brief coverage map](#brief-coverage-map)
- [Repository structure](#repository-structure)
- [How to run](#how-to-run)
- [Testing results](#testing-results)
- [Testing work conclusion](#testing-work-conclusion)
- [Credits and license](#credits-and-license)

## Functional context of the app
**Ride-management system.** Passenger transport company with a varied fleet (Auto/Combi/Moto), permanent/temporary drivers, and registered clients; resources grow with successive sign-ups.

**Users**: 1 administrator and multiple clients.

**Administrator**
- Create drivers and vehicles.
- Listings: clients, drivers, vehicles, trips; driver score/salary; total salaries.
- Create a trip by assigning available driver and vehicle to a pending request.

**Client**
- User registration (no duplicate usernames).
- Trip request: acceptance or rejection when no suitable vehicle (capacity, trunk, pet) is available.
- Pay and rate the driver; view own trip history.

**Key contracts**
- No duplicates for users/vehicles/drivers.
- Maximum 10 passengers per request.
- Vehicle selection by priority and fitness (type/capacity/trunk/pet).
- Fare calculation by zone, luggage, pets, and km; salaries by driver type (gross/net).

## TP context
- **Enunciado** (`EnunciadoTP.docx`): single administrator (“admin”/“admin”) and multiple clients; create/list drivers, vehicles, and trips; create/accept requests; compute salaries and driver ratings; no deletions.
- **Pautas** (`PautasTP.docx`): Maven project with black/white-box tests (cover missing lines), OO integration, persistence (read/write with existing or missing file), GUI, exceptions, and conclusions.
- **SRS** (`SRS.docx`): FR/NFR for users, vehicles, requests, trips; authentication; fare calculation; salaries; binary persistence; no duplicates; max 10 passengers.
- **View behavior** (`Comportamiento_Vistas.docx`): enable/disable rules and messages for Login, Register, Client, and Admin screens (GUI tests follow these).
- **Test worksheets**: `Modelo de Datos - Tabla de Particiones y Bateria de Prueba.xlsx` and `_Persistencia - Tabla de Particiones y Bateria de Prueba.xlsx` define equivalence classes and boundary values used in tests.

## System under test
- **Actors**: 1 admin and several clients.
- **Admin**: create drivers/vehicles, list clients/drivers/vehicles/trips, total salaries, trips per driver, create trip by assigning driver and vehicle.
- **Client**: register, request trip (accept/reject for lack of vehicle), pay and rate, history.
- **Modules**: `modeloDatos` (Administrador, Cliente, Chofer Permanente/Temporario, Vehiculo Moto/Auto/Combi, Pedido, Viaje), `modeloNegocio.Empresa` (singleton), `controlador.Controlador`, `vista.Ventana` + `IOptionPane`.

## Testing strategy
- **Design**: equivalence classes and boundary values from the worksheets for each constructor/method (non-null license plate, seats in range, valid zones, persistence file existing/missing/null, etc.).
- **Reusable scenarios** (business): `EscenarioBase` (empty) and `Escenario1–5` for clients with no trips, pending requests, in-progress trips, and completed trips. Persistence: missing file, empty file, and file with data.
- **Scope**: public methods per contract; generated getters/setters omitted. Messages and exceptions validated (`Mensajes`, `Constantes`). GUI/persistence isolated from external infra.

## Types of tests and suites (with examples from the repo)
- **Unit – data model** (`TP/src/test/java/modelo/dato`)
  - `AutoTest_ConMascota` and `_SinMascota`: score 80/60 depending on trunk; null if over capacity or pet not allowed.
  - `MotoTest`: 1000 points only for 1 pax without trunk/pet; null in invalid cases.
  - `CombiTest_ConMascota/_SinMascota`: 10 * pax with/without 100 extra for trunk; null if over capacity.
  - `ViajeTest_Esc1/2/3`: fare by zone + trunk + pet; finish and rating.
  - `ChoferPermanenteTest/ChoferTemporarioTest`: gross salary by seniority/children; net salary 86%.
- **Unit – business** (`TP/src/test/java/modelo/negocio`)
  - `EmpresaTestEscenarioVacio`: valid inserts and duplicates (`UsuarioYaExiste`, `VehiculoRepetido`), login of non-existent user, total salary with no drivers.
  - `EmpresaTestEscenario1`: correct login/logout and wrong password (message and data in exception).
  - `EmpresaTestEscenario2–5`: `agregarPedido` (no vehicle available, client with request/trip), `vehiculosOrdenadosPorPedido` (ordering/filtering), `crearViaje` (driver/vehicle unavailable or invalid), `calificacionDeChofer`, `getTotalSalarios`.
- **Integration (Controller + mocked View)** (`TP/src/test/java/controlador`)
  - `ControladorTest1` (empty company) and `ControladorTest2` (populated lists) using Mockito on `IVista` and fake `OptionPane`.
  - Flows: correct/incorrect login, registration (passwords mismatch, duplicate user), driver/vehicle creation, requests (no vehicle, client with request/trip), trip creation (missing request, driver/vehicle unavailable, invalid vehicle), rate/pay with and without trip.
- **Persistence** (`TP/src/test/java/persistencia`)
  - `PersistenciaBinTest_Esc1/2/3`: open/close input-output, write/read drivers, vehicles, clients, requests, in-progress/completed trips, and logged user. Cases: missing file, null, no permissions, non-serializable object.
  - `EmpresaDTOTest`: conversion Empresa ↔ EmpresaDTO and collection consistency.
- **Mocks (Mockito) and “MOC”**
  - Mockito stubs for `IVista`; `OptionPane`/`FalseOptionPanel` capture messages without real dialogs.
  - “MOC” in the course refers to using doubles/mocks to isolate dependencies (implemented here with Mockito and fake OptionPane).
- **GUI tests (Robot)** (`TP/src/test/java/Vista/testGUI`)
  - Suite `AllTestGui`: enable/disable in Login/Register, duplicate driver/vehicle creation (`Mensajes.CHOFER_YA_REGISTRADO`, `Mensajes.VEHICULO_YA_REGISTRADO`), trip creation from Admin panel, and request/pay flow in Client panel per `Comportamiento_Vistas.docx`.

## Key scenarios and test data
- `EscenarioBase`: empty company (base).
- `Escenario1`: clients without requests/trips.
- `Escenario2`: clients + drivers + vehicles without requests (ordering vehicles, inserts).
- `Escenario3`: multiple pending requests (request and availability exceptions).
- `Escenario4`: trips in progress (occupied driver/vehicle).
- `Escenario5`: finished and rated trips (averages and salaries).
- Persistence: `empresa.bin` missing, empty, or with data; null open/close; write/read serializable objects.

## Brief coverage map
| Functional requirement | Relevant tests |
| --- | --- |
| Driver creation | `controlador.ControladorTest1#testNuevoChofer`, `controlador.ControladorTest2#testNuevoChofer`, GUI `TestGuiAdmEsc2` |
| Vehicle creation | `ControladorTest1#testNuevoVehiculo`, `ControladorTest2#testNuevoVehiculo`, GUI `TestGuiAdmEsc2` |
| Login/Register | `ControladorTest1#testLogin`, `ControladorTest2#testLogin/testLogin2`, `ControladorTest1#testRegistrar/testRegistrar2`, GUI `AllTestGui` |
| Request with no suitable vehicle | `ControladorTest2#testNuevoPedido2`, `EmpresaTestEscenario3` |
| Request with existing request/trip | `ControladorTest2#testNuevoPedido3/testNuevoPedido4`, `EmpresaTestEscenario3/4/5` |
| Create trip (driver/vehicle available) | `ControladorTest2#testNuevoViaje6`, `EmpresaTestEscenario4/5` |
| Create trip with errors (missing request, driver/vehicle unavailable/invalid) | `ControladorTest2#testNuevoViaje`, `testNuevoViaje2/3/4` |
| Fare calculation | `ViajeTest_Esc1/2/3` |
| Driver salaries | `EmpresaTestEscenarioVacio#testGetTotalSalarios_*` |
| Persistence read/write | `PersistenciaBinTest_Esc1/2/3`, `EmpresaDTOTest` |

## Repository structure
- Docs and spec: `EnunciadoTP.docx`, `PautasTP.docx`, `SRS.docx`, `Comportamiento_Vistas.docx`, `Informe.docx`.
- Worksheets: `Modelo de Datos - Tabla de Particiones y Bateria de Prueba.xlsx`, `_Persistencia - Tabla de Particiones y Bateria de Prueba.xlsx`.
- Code and tests: `TP/src/test/java/` in `controlador`, `modelo/dato`, `modelo/negocio`, `persistencia`, `Vista/testGUI`.
- Build/deps: `TP/pom.xml` (JUnit 4.13.2, Mockito 3.2.4, `lib/SubiQueTeLlevo.jar` as `system` dependency), sample binaries `TP/datos.bin`, `TP/empresa.bin`.
- Reports: `ResultadosTesteo.pdf` and `TP/target/site/surefire-report.html` (after `mvn test`).

## How to run
Requirements: Java 8+ and Maven.

```bash
# From repo root
cd TP-TallerProgramacio1/TP
mvn test
```
- Specific suites (exact names):  
  - `mvn -Dtest=AllTest test` (root suite)  
  - `mvn -Dtest=controlador.AllTestController test`  
  - `mvn -Dtest=modelo.negocio.EmpresaTestSuite test`  
  - `mvn -Dtest=persistencia.AllTestPersistencia test`  
- Skip GUI (headless) or use Xvfb:  
  `mvn -Dtest=AllTest -Dsurefire.excludes=Vista/testGUI/** test`
- If Robot fails due to focus/timing, run locally, close apps that steal focus, and retry.
- Reports: `TP/target/surefire-reports/` (text) and `TP/target/site/surefire-report.html` (HTML).

## Testing results
- `Empresa.getTotalSalarios` returns amounts lower than expected (permanent/temporary drivers).
- `Empresa.vehiculosOrdenadosPorPedido` does not order by score and returns unfit vehicles.
- `Empresa.agregarPedido` does not throw `SinVehiculoParaPedido`, `ClienteConPedidoPendiente`, or `ClienteConViajePendiente` in several scenarios.
- `Empresa.crearViaje` returns incorrect messages/exceptions (driver not available vs missing request; vehicle unavailable/invalid).
- `Empresa.calificacionDeChofer` does not throw `SinViajesException` and miscalculates the average.
- Persistence: Empresa ↔ EmpresaDTO conversion and binary read/write validated, including missing/empty files.

## Testing work conclusion
- Pautas were met: Maven, unit/integration/persistence/GUI tests, evidence, and scenarios derived from partition tables.
- The contract- and scenario-based approach surfaced defects in business rules and calculations, showing test plan effectiveness.
- For CI/headless, run without GUI or with Xvfb; to reproduce Robot issues, run locally with stable focus.

## Authors
- Criado, Facundo
- Nieto, Iván Ezequiel 
- Saladino, Juan Cruz
- San Pedro, Gianfranco
