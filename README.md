# triggers-y-funciones
Triggers y funciones  ejercicio # 10 extracción minera 
#scrip 


use mineria; 
# TRIGGERS

DELIMITER //

CREATE TABLE AUDITORIA_EXTRACCION (
    id INT AUTO_INCREMENT PRIMARY KEY,
    id_extraccion INT,
    fecha DATETIME,
    accion VARCHAR(50)
);
//

CREATE TRIGGER trg_insert_extraccion
AFTER INSERT ON EXTRACCION
FOR EACH ROW
BEGIN
    INSERT INTO AUDITORIA_EXTRACCION (id_extraccion, fecha, accion)
    VALUES (NEW.id_extraccion, NOW(), 'INSERT');
END;
//
DELIMITER ; 


DELIMITER //

CREATE TRIGGER trg_validar_extraccion
BEFORE INSERT ON EXTRACCION
FOR EACH ROW
BEGIN
    IF NEW.cantidad <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cantidad inválida';
    END IF;
END;
//
DELIMITER ;


DELIMITER //

CREATE TRIGGER trg_mantenimiento_maquinaria
BEFORE UPDATE ON MAQUINARIA
FOR EACH ROW
BEGIN
    IF NEW.horas_uso > 5000 THEN
        SET NEW.estado = 'Mantenimiento urgente';
    END IF;
END;
//
DELIMITER ; 

DELIMITER //

CREATE TABLE AUDITORIA_ELIMINACIONES (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tabla VARCHAR(50),
    id_registro INT,
    fecha DATETIME
);
//

CREATE TRIGGER trg_delete_extraccion
BEFORE DELETE ON EXTRACCION
FOR EACH ROW
BEGIN
    INSERT INTO AUDITORIA_ELIMINACIONES (tabla, id_registro, fecha)
    VALUES ('EXTRACCION', OLD.id_extraccion, NOW());
END;
//
DELIMITER ; 

DELIMITER //

CREATE TRIGGER trg_validar_voladura
BEFORE INSERT ON VOLADURA
FOR EACH ROW
BEGIN
    IF NEW.radio_seguridad < 50 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Radio de seguridad muy bajo';
    END IF;
END;
// 

DELIMITER ; 

# COMO COMPROBRAR 

INSERT INTO EXTRACCION (fecha, turno, id_zona, tipo_mineral, cantidad, ley_mineral, horas_trabajadas)
VALUES ('2026-04-20', 'Mañana', 1, 'Oro', 100, 2.5, 8);

#Funciones 

DELIMITER //

CREATE FUNCTION CalcularProduccionTotalZona(p_zona INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);

    SELECT SUM(cantidad)
    INTO total
    FROM EXTRACCION
    WHERE id_zona = p_zona;

    RETURN IFNULL(total, 0);
END //

DELIMITER ;


DELIMITER //

CREATE FUNCTION ObtenerPromedioLeyMineral(p_zona INT)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE promedio DECIMAL(5,2);

    SELECT AVG(ley_mineral)
    INTO promedio
    FROM EXTRACCION
    WHERE id_zona = p_zona;

    RETURN IFNULL(promedio, 0);
END //

DELIMITER ;



DELIMITER //

CREATE FUNCTION CalcularAntiguedadEmpleado(p_empleado INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE anios INT;

    SELECT TIMESTAMPDIFF(YEAR, fecha_contratacion, CURDATE())
    INTO anios
    FROM EMPLEADO
    WHERE id_empleado = p_empleado;

    RETURN IFNULL(anios, 0);
END //

DELIMITER ;


DELIMITER //

CREATE FUNCTION EvaluarEstadoMaquinaria(p_horas INT)
RETURNS VARCHAR(50)
DETERMINISTIC
BEGIN
    DECLARE estado VARCHAR(50);

    IF p_horas < 3000 THEN
        SET estado = 'Óptima';
    ELSEIF p_horas BETWEEN 3000 AND 5000 THEN
        SET estado = 'Mantenimiento preventivo';
    ELSE
        SET estado = 'Mantenimiento urgente';
    END IF;

    RETURN estado;
END //

DELIMITER ;


DELIMITER //

CREATE FUNCTION CalcularRendimientoMineral(
    p_entrada DECIMAL(10,2),
    p_salida DECIMAL(10,2)
)
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
    DECLARE rendimiento DECIMAL(5,2);

    IF p_entrada = 0 THEN
        RETURN 0;
    END IF;

    SET rendimiento = (p_salida / p_entrada) * 100;

    RETURN rendimiento;
END //

DELIMITER ;

# como probrarlas 

SELECT CalcularProduccionTotalZona(1);

SELECT ObtenerPromedioLeyMineral(1);

SELECT CalcularAntiguedadEmpleado(1);

SELECT EvaluarEstadoMaquinaria(4500);

SELECT CalcularRendimientoMineral(1000, 850);

