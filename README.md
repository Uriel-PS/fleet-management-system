# Fleet Management System

Project developed for the Software Engineering course, Computer Science program at UDESC (Universidade do Estado de Santa Catarina). Implements the backend of a vehicle fleet management system, with business rules for refueling, maintenance, vehicle status control, and cost indicator calculation.

**Authors:** Uriel Pacheco de Souza and Kelwin

## Table of Contents

- [Architecture](#architecture)
- [Design Patterns Used](#design-patterns-used)
- [Running the System](#running-the-system)
- [Running the Tests](#running-the-tests)
- [Repository Structure](#repository-structure)
- [Default User (Sample Data)](#default-user-sample-data)
- [Requirements Traceability](#requirements-traceability)

## Architecture

The project follows the **MVC (Model-View-Controller)** pattern:

- **`models/`**: domain classes (pure business rules): `Veiculos` (Vehicles), `Pessoa`/`Motorista`/`Gestor` (Person/Driver/Manager), `Abastecimento` (Refueling), `Manutencao` (Maintenance), `Oficina` (Workshop), and `Dashboard`. No dependency on the interface layer, fully testable in isolation.
- **`patterns/`**: interfaces and classes implementing the **State** and **Observer** design patterns, used by the models.
- **`controllers/`**: the `FrotaController` class, which receives actions from the *view*, triggers the correct *models*, and returns the result. This is also where **role-based access control** rules live (RNF07), such as requiring the logged-in user to be a Manager before registering a vehicle.
- **`views/`**: the `CLI` class (`terminalView.py`), responsible only for displaying menus and capturing keyboard input. Contains no business logic: all decisions are delegated to the controller.

```
View (CLI)  →  Controller (FrotaController)  →  Models (Veiculos, Pessoa, ...)
                                                       ↑
                                              Patterns (State, Observer)
```

## Design Patterns Used

### State

Implemented in `patterns/estados.py`. Controls a vehicle's status (Available, In Operation, In Maintenance) and prevents invalid transitions, in particular the project's core constraint: **a vehicle cannot start an operation while under maintenance**. Each concrete state (`EstadoDisponivel`, `EstadoEmOperacao`, `EstadoEmManutencao`) knows which state it can transition to, and raises `TransicaoInvalidaError` when an invalid transition is requested.

### Observer

Implemented in `patterns/observer.py`. The `Veiculos` class is the `Subject`: it maintains a list of observers and notifies them via `notificar()`. The `Gestor` class implements `Observer` (`atualizar()` method), automatically receiving an alert whenever a vehicle's mileage reaches the threshold for its next scheduled maintenance during a refueling event.

## Running the System

Requirement: Python 3.10 or higher.

From the project root (where `main.py` is located):

```bash
python main.py
```

This launches the system's CLI with a set of sample data already loaded (see [Default User](#default-user-sample-data) below), allowing you to log in and navigate the Manager or Driver menus.

> On Windows, if the `python` command isn't recognized, try `py main.py`.

## Running the Tests

The project uses **pytest**. The only external dependency is pytest itself (and, optionally, `pytest-cov` for coverage reports).

```bash
pip install pytest pytest-cov
```

Run all tests:

```bash
pytest
```

Run with individual test names shown (recommended to follow execution):

```bash
pytest -v
```

Run with code coverage report:

```bash
pytest --cov=models --cov=patterns --cov=controllers --cov-report=term-missing
```

**Current result:** **32 tests**, 100% passing.

| Test File | Count | Covers |
|---|---|---|
| `test_veiculos.py` | 8 | `Veiculos`: State pattern, Observer pattern, odometer update |
| `test_pessoa.py` | 5 | Authentication, roles, Manager receiving alerts |
| `test_dashboard.py` | 7 | Cost-per-km calculation, average fuel consumption, report generation |
| `test_abastecimento.py` | 4 | Creation validation (negative or zero liters/amount) |
| `test_manutencao.py` | 4 | Type and mileage validation |
| `test_oficina.py` | 2 | Registration factory and ID generation |
| `test_frotacontroller.py` | 2 | Login and blocking of Manager-restricted actions |
| **Total** | **32** | |

> **Note on coverage:** `models/` and `patterns/` (where the business logic and design patterns live) have coverage between 67% and 100%, concentrated on the most critical methods, indicator calculation (`Dashboard`), state transitions, and alert triggering. `controllers/frotaController.py` has lower coverage (~52%), since current tests focus the controller on the login flow and permission blocking (RNF07); most other controller methods are direct pass-throughs to already individually tested models.

## Repository Structure

```
gestao_frotas/
├── main.py                       # entry point: launches the CLI
├── pytest.ini                    # pytest configuration
├── conftest.py                   # (empty, required for pytest to locate the root)
├── models/
│   ├── veiculos.py                # Veiculos class (Observer Subject)
│   ├── pessoa.py                  # Pessoa, Motorista, Gestor (Observer)
│   ├── abastecimento.py
│   ├── manutencao.py
│   ├── oficina.py
│   └── dashboard.py                # cost-per-km and average consumption calculation
├── patterns/
│   ├── estados.py                  # State pattern
│   └── observer.py                 # Observer pattern
├── controllers/
│   └── frotaController.py         # control layer (MVC)
├── views/
│   └── terminalView.py            # CLI (RNF04)
└── tests/
    ├── conftest.py                 # shared test fixtures
    ├── test_veiculos.py
    ├── test_pessoa.py
    ├── test_dashboard.py
    ├── test_abastecimento.py
    ├── test_manutencao.py
    ├── test_oficina.py
    └── test_frotacontroller.py
```

## Default User (Sample Data)

When the system starts (`python main.py`), the following data is automatically created to facilitate manual testing:

- **Default Manager:** login `admin`, password `admin123`
- **Sample Driver:** login `joao`, password `123`
- **Sample Workshop:** "Mecânica Diesel Express"
- **Three sample vehicles**, already configured to demonstrate:
  - `ABC-1234`: normal cost-per-km and fuel consumption calculation (5 km/L expected)
  - `DEF-5678`: already in "Under Maintenance" state (to test the State pattern)
  - `XYZ-9876`: a refueling event that exceeds the maintenance threshold, triggering the automatic alert (to test the Observer pattern)

## Requirements Traceability

| Requirement | Where It's Implemented |
|---|---|
| RF01 – Vehicle Management | `Veiculos.editar()`, `Veiculos.inativar()`, `FrotaController.cadastrarVeiculo()` |
| RF02 – Vehicle State Control | `patterns/estados.py` (State pattern) |
| RF03 – Refueling Records | `Abastecimento`, `Veiculos.registrarAbastecimento()` |
| RF04 – Maintenance Records | `Manutencao`, `Veiculos.registrarManutencao()` |
| RF05 – Alert System | `patterns/observer.py`, `Gestor.atualizar()` (Observer pattern) |
| RF06 – Indicator Calculation (Dashboard) | `Dashboard.calcularCustoPorKm()`, `calcularMediaConsumo()`, `gerarRelatorio()` |
| RF07 – User Management | `FrotaController.cadastrarMotorista()`, `cadastrarGestor()`, `inativarUsuario()` |
| RF08 – System Authentication | `Pessoa.autenticar()`, `FrotaController.login()` |
| RF09 – Partner Workshop Registration | `Oficina`, `FrotaController.cadastrarOficina()` |
| RNF04 – Command-Line Interface | `views/terminalView.py` |
| RNF05 – State Transition Restriction | `patterns/estados.py` (`TransicaoInvalidaError`) |
| RNF06 – Automatic Odometer Update | `Veiculos.registrarAbastecimento()` |
| RNF07 – Role-Based Access Control | `FrotaController._exigir_gestor()`, `Pessoa.perfil()` |

## Documentation

The full academic report (requirements specification, in Portuguese) is available in this repository as a PDF.
