# DataBase-AlkeWallet
Base de datos para Alke Wallet
AlkeWallet
README — Base de Datos MySQL
Evaluación Módulo 3  |  Denisse Ibacache Montenegro  |  Grupo 1
1. Descripción del Proyecto
AlkeWallet es una base de datos relacional desarrollada en MySQL que simula el backend de una billetera digital (digital wallet). Permite gestionar usuarios, tipos de moneda y transacciones de transferencia entre cuentas, cumpliendo con los requisitos de la Evaluación del Módulo 3.

2. Estructura de la Base de Datos
La base de datos AlkeWallet contiene tres tablas principales relacionadas entre sí mediante claves foráneas (FOREIGN KEY):

2.1 Tabla: moneda
Almacena los tipos de moneda disponibles en el sistema.

Campo	Tipo	Restricciones	Descripción
currency_id	INT	PK, AUTO_INCREMENT, NOT NULL	Identificador único de la moneda
currency_name	VARCHAR(100)	NOT NULL	Nombre de la moneda (ej: Peso chileno)
currency_symbol	VARCHAR(50)	NOT NULL	Símbolo de la moneda (ej: CLP)

2.2 Tabla: usuario
Almacena la información de los usuarios registrados en el sistema.

Campo	Tipo	Restricciones	Descripción
user_id	INT	PK, AUTO_INCREMENT, NOT NULL	Identificador único del usuario
username	VARCHAR(100)	NOT NULL, UNIQUE	Nombre de usuario
email	VARCHAR(150)	NOT NULL, UNIQUE	Correo electrónico
password	VARCHAR(150)	NOT NULL	Contraseña del usuario
balance	DECIMAL(10,2)	NOT NULL, DEFAULT 0.00	Saldo disponible
currency_id	INT	FK → moneda(currency_id)	Tipo de moneda asignado
created_user_date	DATETIME	DEFAULT CURRENT_TIMESTAMP	Fecha de creación del registro

2.3 Tabla: transaccion
Registra cada transferencia realizada entre dos usuarios del sistema.

Campo	Tipo	Restricciones	Descripción
transaction_id	INT	PK, AUTO_INCREMENT, NOT NULL	Identificador único de la transacción
sender_user_id	INT	NOT NULL, FK → usuario(user_id)	Usuario que envía los fondos
receiver_user_id	INT	NOT NULL, FK → usuario(user_id)	Usuario que recibe los fondos
amount	DECIMAL(10,2)	NOT NULL, CHECK(amount > 0)	Monto transferido
transaction_date	DATETIME	DEFAULT CURRENT_TIMESTAMP	Fecha y hora de la transacción

3. Creación de la Base de Datos
Ejecutar los siguientes comandos SQL en MySQL Workbench o cliente equivalente:

CREATE DATABASE AlkeWallet;
USE AlkeWallet;

-- Tabla Moneda
CREATE TABLE Moneda (
    currency_id     INT           AUTO_INCREMENT PRIMARY KEY,
    currency_name   VARCHAR(100)  NOT NULL,
    currency_symbol VARCHAR(50)   NOT NULL
);

-- Tabla Usuario
CREATE TABLE Usuario (
    user_id           INT           AUTO_INCREMENT PRIMARY KEY,
    username          VARCHAR(100)  NOT NULL UNIQUE,
    email             VARCHAR(150)  NOT NULL UNIQUE,
    password          VARCHAR(150)  NOT NULL,
    balance           DECIMAL(10,2) NOT NULL DEFAULT 0.00,
    currency_id       INT,
    FOREIGN KEY(currency_id) REFERENCES Moneda(currency_id)
);

-- Columna de fecha de creacion (agregada con ALTER TABLE)
ALTER TABLE usuario
ADD COLUMN created_user_date DATETIME DEFAULT current_timestamp;

-- Tabla Transaccion
CREATE TABLE Transaccion (
    transaction_id   INT            AUTO_INCREMENT PRIMARY KEY,
    sender_user_id   INT            NOT NULL,
    receiver_user_id INT            NOT NULL,
    amount           DECIMAL(10,2)  NOT NULL CHECK(amount > 0),
    transaction_date DATETIME       DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY(sender_user_id)   REFERENCES Usuario(user_id),
    FOREIGN KEY(receiver_user_id) REFERENCES Usuario(user_id)
);

4. Registros de Prueba
4.1 Inserción de Monedas
INSERT INTO moneda (currency_name, currency_symbol) VALUES
    ('Peso chileno',   'CLP'),
    ('Dolar',          'USD'),
    ('Peso argentino', 'ARS'),
    ('Peso mexicano',  'MXN');

4.2 Inserción de Usuarios
INSERT INTO usuario (username, email, password, balance) VALUES
    ('Ana Rojas',           'ana.rojas@gmail.com',           'anarojas1',    15000.00),
    ('Alberto Contreras',   'alberto.contreras@gmail.com',   'alberto1980',  64500.00),
    ('Sebastian Cisteneras','seba.ciste@gmail.com',          'cisternas.se', 36400.00),
    ('Constanza Martinez',  'martinez.coni@gmail.com',       'martinez1991', 85500.00),
    ('Karla Soto',          'soto.k@gmail.com',              'k.soto89',     62480.00),
    ('Pablo Gomez',         'p.gomez@gmail.com',             'pablogomez1',  43200.00),
    ('Francisca Hidalgo',   'Fran.hidalgo@gmail.com',        'hidalgofran',  26700.00);

4.3 Asignación de Tipo de Moneda (FOREIGN KEY)
Ejemplo: asignar Peso Chileno (currency_id = 1) al usuario con user_id = 1:
UPDATE Usuario SET currency_id = 1 WHERE user_id = 1;

5. Ejemplo de Transacción
Transferencia de $10.000 desde user_id 1 (Ana Rojas) hacia user_id 2 (Alberto Contreras). Se utiliza una transacción atómica para garantizar integridad:

START TRANSACTION;

-- Registrar la transaccion
INSERT INTO transaccion (sender_user_id, receiver_user_id, amount)
VALUES (1, 2, 10000.00);

-- Descontar saldo al emisor
UPDATE usuario SET balance = balance - 10000.00 WHERE user_id = 1;

-- Agregar saldo al receptor
UPDATE usuario SET balance = balance + 10000.00 WHERE user_id = 2;

COMMIT;

Resultado tras la transacción:

Usuario	Balance Inicial	Balance Final
Ana Rojas (user_id 1)	$15.000,00	$5.000,00
Alberto Contreras (user_id 2)	$64.500,00	$74.500,00

6. Consultas SQL Principales
6.1 Ver todas las tablas de la BD
SHOW TABLES;

6.2 Explorar estructura de una tabla
DESCRIBE transaccion;
DESCRIBE usuario;
DESCRIBE moneda;

6.3 Obtener la moneda elegida por un usuario
SELECT u.username, m.currency_name
FROM Usuario u
JOIN Moneda m ON u.currency_id = m.currency_id
WHERE u.username = 'Ana Rojas';

6.4 Ver todas las transacciones
SELECT * FROM transaccion;

6.5 Transacciones de un usuario específico
SELECT * FROM Transaccion
WHERE sender_user_id = 1;

6.6 Modificar el email de un usuario
UPDATE Usuario
SET email = 'pablo.g@hotmail.com'
WHERE user_id = 6;

7. Diagrama de Relaciones
Las relaciones entre tablas son las siguientes:
•	moneda (1) ←→ (N) usuario: un tipo de moneda puede ser asignado a muchos usuarios.
•	usuario (1) ←→ (N) transaccion (sender_user_id): un usuario puede enviar múltiples transferencias.
•	usuario (1) ←→ (N) transaccion (receiver_user_id): un usuario puede recibir múltiples transferencias.

8. Requisitos
•	MySQL 8.0 o superior
•	MySQL Workbench (u otro cliente SQL compatible)

9. Información del Proyecto
Autora	Denisse Ibacache Montenegro
Grupo	Grupo 1
Módulo	Evaluación Módulo 3
Tecnología	MySQL / MySQL Workbench


