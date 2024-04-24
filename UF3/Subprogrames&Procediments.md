# EXERCICIS SUBPROGRAMES
* Aquests exercicis estan realitzats amb la base de dades de rrhh.

### EXERCICI 1
- Fes una funció anomenada spData, tal que donada una data en format
MySQL ( AAAA-MM-DD ) ens retorni una cadena de caràcters en format DD-MM-AAAA
Exemple : SELECT spData('1988-12-01') => 01-12-1988:
```mysql
 DELIMITER //
DROP FUNCTION IF EXISTS spData //
CREATE FUNCTION spData(pData DATE) RETURNS CHAR(10)
DETERMINISTIC
BEGIN
	DECLARE vRetorn CHAR(10);
    
    SET vRetorn = date_format(pData, "%d-%m-%Y");
    
    RETURN vRetorn;
END
//
DELIMITER;
```
### EXERCICI 2
- Fes una funció anomenada spPotencia, tal que donada una base i un
exponent, ens calculi la seva potència. Intenta no utilitzar la funció POW.
Exemple : SELECT spPotencia(2,3) => 8:
``` mysql
DELIMITER //
DROP FUNCTION IF EXISTS spPotencia //
CREATE FUNCTION spPotencia(pBase INT, pExp INT) RETURNS BIGINT
DETERMINISTIC
BEGIN
	DECLARE vRetorn BIGINT;
    
    IF pBase IS NOT NULL AND pExp IS NOT NULL THEN
		SET vRetorn = 1;
        WHILE pExp > 0 DO
			SET vRetorn = vRetorn * pBase;
            SET pExp = pExp -1;
		END WHILE;
        END IF;
    
    RETURN vRetorn;
END
//
DELIMITER;
```
### EXERCICI 3
-Fes una funció anomenada spIncrement que donat un codi d’empleat i un
% de increment, ens calculi el salari sumant aquest percentatge.

Per exemple, suposem que l’ empleat amb id_empleat = 124 té un salari de 1000
Exemple: SELECT spIncrement(124,10) obtindriem -> 1100:
``` mysql
DELIMITER //
DROP FUNCTION IF EXISTS spIncrement //
CREATE FUNCTION spIncrement(pEmpleatId INT, pIncrement FLOAT) RETURNS FLOAT
NOT DETERMINISTIC READS SQL DATA
BEGIN
	DECLARE vRetorn FLOAT;
    
    SELECT salari*(pIncrement + 100) /100 INTO vRetorn
		FROM empleats
	WHERE empleat_id = pEmpleatId;
    
    RETURN vRetorn;
END
//
DELIMITER;
```
### EXERCICI 4
- Fes una funció anomenada spPringat, tal que li passem un codi de
departament, i ens torni el codi d’empleat que guanya menys d’aquell departament:
``` mysql

DELIMITER //
DROP FUNCTION IF EXISTS spPringat //
CREATE FUNCTION spPringat(pDepartamentId INT) RETURNS INT
NOT DETERMINISTIC READS SQL DATA
BEGIN
	DECLARE vRetorn  INT;
    
    SELECT empleat_id INTO vRetorn
		FROM empleats
	WHERE departament_id = pDepartamentId
    ORDER BY pDepartamentId
    LIMIT 1;
    
    RETURN vRetorn;
END
//
DELIMITER;
```
### EXERCICI 5
- Utilitzant la funció spPringat fes una consulta per obtenir de cada
departament, l’empleat pringat. Mostra el codi i nom del departament, i el codi d’empleat:
``` mysql
SELECT rrhh.spPringat();
```
### EXERCICI 6
- Fes una funció anomenada spCategoria, tal que donat un codi d’empleat,
ens digui en quina categoria professional està. El criteri que volem seguir per determinar
la categoria professional és en funció dels anys que porta treballant a l’empresa:
Entre 0 i 1 anys -> Auxiliar
Entre 2 i 10 anys -> Oficial de Segona
Entre 11 i 20 Anys -> Oficial de Primera
Més de 20 anys -> Que es jubili!:
``` mysql
DELIMITER //
DROP FUNCTION IF EXISTS spCategoria //
CREATE FUNCTION spCategoria(pCodiEmpleat INT) RETURNS VARCHAR(20)
NOT DETERMINISTIC READS SQL DATA
BEGIN
	DECLARE vRetorn VARCHAR(20);
    DECLARE anys INT;
    
    SELECT YEAR(CURDATE()) - YEAR(data_contractacio) INTO anys
    FROM empleats
    WHERE empleat_id = pCodiEmpleat;
    
    SET vRetorn = CASE
	WHEN anys BETWEEN 0 AND 1 THEN "Auxiliar"
    WHEN anys BETWEEN 2 AND 10 THEN "Oficial de Segona"
    WHEN anys BETWEEN 11 AND 20 THEN "Oficial de Primera"
    ELSE "Que es jubili"
    END;
    
    RETURN vRetorn;
END //
DELIMITER ;
```
### EXERCICI 7
- Fes una consulta utilitzant la funció anterior perquè mostri mostri de cada
empleat, el codi d’empleat, el nom, els anys treballats i la categoria professional a la que
pertany:
``` mysql
SELECT empleat_id, nom, cognoms, spCategoria(empleat_id)
	FROM empleats;

```
### EXERCICI 8
- Fes una funció anomenada spEdat, tal que donada una data per paràmetre
ens retorni l'edat d'una persona. Les dates posteriors a la data d'avui han de retornar 0:
``` mysql
DELIMITER //
DROP FUNCTION IF EXISTS spEdat //
CREATE FUNCTION spEdat(pDataIntroduida DATE) RETURNS INT UNSIGNED
NOT DETERMINISTIC READS SQL DATA
BEGIN
	DECLARE vRetorn INT UNSIGNED;
    
    SET vRetorn = CASE
		WHEN pDataIntroduida >= CURDATE() THEN 0
        ELSE YEAR(CURDATE()) - YEAR(pDataIntroduida) - (RIGHT(CURDATE(), 5) < RIGHT(pDataIntroduida, 5))
        END;
        RETURN vRetorn;
	END // 
DELIMITER ;
```
### EXERCICI 9
- Fes una funció que ens retorni el número de directors (caps) diferents tenim:
``` mysql
DELIMITER // 
DROP FUNCTION IF EXISTS spNumDirectors //
CREATE FUNCTION spNumDirectors() RETURNS INT
NOT DETERMINISTIC READS SQL DATA
BEGIN
	DECLARE vRetorn INT;
    
    SELECT COUNT(DISTINCT empleat_id) INTO vRetorn
		FROM empleats
	WHERE feina_codi = 'AD_PRES';
    
	RETURN vRetorn;
END //
DELIMITER ;
```
### EXERCICI 10
- Quina instrucció utilitzarem si volem veure el contingut de la funció
spPringat?:
``` mysql
SHOW CREATE FUNCTION spPringat;

SELECT * FROM information_schema.routines;
	

SHOW TABLES FROM information_schema;

```

# EXERCICIS PROCEDIMENTS
### EXERCICI 1
- Fes un procediment que permeti obtenir la data i hora del sistema i l’usuari
actual:
``` mysql
DELIMITER // 

DROP PROCEDURE IF EXISTS sp_DadesActuals //
CREATE PROCEDURE sp_DadesActuals()
BEGIN
	SELECT NOW() AS 'Data i Hora Actuals',
    USER() AS 'Usuari Actual';
END //
DELIMITER ; 
```
### EXERCICI 2
- Fes un procediment que intercanvii el sou de dos empleats passats per
paràmetre:
``` mysql
DELIMITER //
DROP PROCEDURE IF EXISTS sp_IntercanviSalarial //
CREATE PROCEDURE sp_IntercanviSalarial(IN pCodiEmpleat1 INT, IN pCodiEmpleat2 INT)
BEGIN 
    DECLARE vSalari1 DECIMAL(8,2);
    
    IF ( SELECT empleat_id
			FROM empleats 
		WHERE empleat_id = pCodiEmpleat1) IS NOT NULL
        AND
        (SELECT empleat_id
			FROM empleats 
		WHERE empleat_id = pCodiEmpleat2) IS NOT NULL 
        THEN
    SELECT salari INTO vSalari1 FROM empleats WHERE empleat_id = pCodiEmpleat1;
   
    UPDATE empleats SET salari = (SELECT salari 
									FROM empleats
								WHERE empleat_id = pCodiEmpleat2)
					WHERE empleat_id = pCodiEmpleat1;
                    
    UPDATE empleats SET salari = vSalari1 WHERE empleat_id = pCodiEmpleat2;
    END IF;
END //
DELIMITER ;
```
### EXERCICI 3
- Fes un procediment que donat dos Ids d'empleat assigni el codi de
departament del primer en el segon:
``` mysql
DELIMITER //
DROP PROCEDURE IF EXISTS sp_CanviarDep //
CREATE PROCEDURE sp_CanviarDep(pCodiEmpleat1 INT, pCodiEmpleat2 INT)
BEGIN
	DECLARE vDepartament1 INT;
    
    SELECT departament_id INTO vDepartament1 FROM empleats WHERE codi_empleat = pCodiEmpleat1;
    
	UPDATE empleats SET departament_id = vDepartament1 WHERE codi_empleat = pCodiEmpleat2;
END //
DELIMITER ;
```
### EXERCICI 4
- Fes un procediment que donat dos codis de departament assigni tots els
empleats del segon en el primer. Un cop executat el procediment el departament que
correspont en el segon paràmetre ha de quedar desert/sense cap empleat:
``` mysql
DELIMITER //
DROP PROCEDURE IF EXISTS sp_DepExchange //
CREATE PROCEDURE sp_DepExchange(IN pCodiDepartament1 INT, IN pCodiDepartament2 INT)
BEGIN
	UPDATE empleats
		SET departament_id = pCodiDepartament1
        WHERE departament_id = pCodiDepartament2;
    
END //
DELIMITER ;
```
### EXERCICI 5
- Fes un procediment per mostrar un llistat dels empleats. Volem veure el
id_empleat, nom_empleat, nom_departament i el nom de la localització del departament:
``` mysql
DELIMITER //
DROP PROCEDURE IF EXISTS sp_MostrarEmp //
CREATE PROCEDURE sp_MostrarEmp(pCodiEmpleat INT)
BEGIN 
	SELECT emp.empleat_id, emp.nom, dp.nom, dp.localitzacio_id 
		FROM empleats emp
	INNER JOIN departaments dp ON emp.departament_id = dp.departament_id
    WHERE emp.empleat_id = pCodiEmpleat;
END //
DELIMITER ; 
```
### EXERCICI 6
- Fes un procediment que donat un codi d’empleat, ens doni la informació de
l’empleat ( agafa la informació que creguis rellevant):
``` mysql
DELIMITER //
DROP PROCEDURE IF EXISTS sp_DadesRellevantsEmp //
CREATE PROCEDURE sp_DadesRellevantsEmp(pCodiEmpleat INT)
BEGIN
	SELECT *
		FROM empleats
	WHERE empleat_id = pCodiEmpleat;
END //
DELIMITER ;

CALL sp_DadesRellevantsEmp(100);
```

### EXERCICI 7
- Volem fer un registre dels usuaris que entren al nostre sistema. Per fer-ho
primer caldrà crear una taula amb dos camps, un per guardar l’usuari i l’altre per guardar
la data i hora de l’accés. 
``` mysql
DROP PROCEDURE IF EXISTS sp_RegistreUsuari()
DELIMITER // 
CREATE PROCEDURE sp_RegistreUsuari()
BEGIN 
	INSERT INTO registre_usuaris(usuari,acces)
		VALUES(CURRENT_USER(), NOW());
END //
```

### EXERCICI 10
- Fes un procediment que donat un codi d’empleat, ens posi en paràmetres
de sortida el nom i el cognom. Indica com ho pots fer per comprovar si el procediment et
funciona.
``` mysql
DROP PROCEDURE IF EXISTS sp_DadesEmpleat()
DELIMITER //
CREATE PROCEDURE sp_DadesEmpleat(IN pEmpleatId INT, OUT pEmpleatNom VARCHAR(20), OUT pEmpleatCognom VARCHAR(25))
BEGIN
	SELECT nom, cognoms INTO pEmpleatNom, pEmpleatCognoms
		FROM empleats
	WHERE empleat_id = pEmpleatId;
END //
```

### EXERCICI 11
 - Fes un procediment que ens permeti modificar el nom i cognom d’un
empleat. 
``` mysql
DROP PROCEDURE IF EXISTS sp_ModificarDadesEmpleat
DELIMITER //
CREATE PROCEDURE sp_ModificarDadesEmpleat(IN pEmpleatId INT, OUT pEmpleatNom VARCHAR(20), OUT pEmpleatCognom VARCHAR(25))
BEGIN
	UPDATE empleats
    SET nom = pEmpleatNom,
		cognoms = pEmpleatCognoms
	WHERE empleat_id = pEmpleatId;
END //
```

### EXERICIC 12
 - Crea una taula d’auditoria anomenada logs_usuaris. Aquesta taula la
utilitzarem per monitoritzar algunes de les accions que fan els usuaris sobre les dades, per exemple si actualitzen dades, eliminen registres. La taula ha de tenir els següents camps:

usuari | VARCHAR(100) | Usuari que ha realitzat l’acció
data | DATETIME | Data-Hora en que s’ha realitzat l’acció
taula | VARCHAR(50) | Taula sobre la que es realitza l’acció
accio | VARCHAR(20) | “ELIMINAR”,”AFEGIR”,”MODIFICAR”,”INSERIR”
valor_pk | VARCHAR(200) | valor que identifica el registre que ha patit l’acció

Fes un procediment amb nom spRegistrarLog que rebrà com a paràmetres el nom de la
taula, l’acció i el valor_pk.
Aquest procediment només cal que insereixi un registre en la taula logs_usuaris amb les dades rebudes, tenint en compte l’usuari actual i la data-hora del sistema.
``` mysql
CREATE TABLE logs_usuaris(
	usuari 		VARCHAR(100),
    data		DATETIME,
    taula, 		VARCHAR(50),
    accio		VARCHAR(20),
    valor_pk	VARCHAR(200)
);

DROP PROCEDURE IF EXISTS sp_RegistrarLog;
DELIMITER //
CREATE PROCEDURE sp_RegistrarLog(pTaula VARCHAR(50), pAccio VARCHAR(20), pValorPK VARCHAR(200))
BEGIN
	INSERT INTO logs_usuaris (usuari,data,taula,accio,valor_pk)
		VALUES(CURRENT_USER(),NOW(), pTaula, pAccio, pValorPK);
END //
```

### EXERCICI 13
- Fes un procediment que ens permeti eliminar un departament determinat. El departament s’ha d’eliminar de la taula departaments. Utilitza a més dins d’aquest
procediment, el procediment creat anteriorment (spRegistrarLog) per guardar també un
registre del que ha realitzat l’usuari. Ens ha de quedar clar que l’usuari actual, en data X ha eliminat de la taula DEPARTAMENTS el codi departament Y.

Per exemple, si fem una crida al procediment per eliminar un departament: CALL eliminarDept(300); El departament 300 s’ha d’eliminar de la taula departaments, i a més si fem una SELECT de la taula logs_usuaris, veuriem per exemple el següent:

CALL eliminarDept(300);
El departament 300 s’ha d’eliminar de la taula departaments, i a més si fem una SELECT
de la taula logs_usuaris, veuriem per exemple el següent:

``` mysql
DROP PROCEDURE IF EXISTS sp_EliminarDep;
DELIMITER //
CREATE PROCEDURE sp_EliminarDep(pDepId INT)
BEGIN
	DECLARE vDepId INT;
	SELECT departament_id INTO vDepId
		FROM departaments
	WHERE departament_id = pDepId;
    
IF vDepId IS NOT NULL THEN
	DELETE FROM  departaments
		WHERE departament_id = pDepId;
        
	DELETE FROM empleats
		WHERE departament_id = pDepID;
        
END IF;
END //
```









