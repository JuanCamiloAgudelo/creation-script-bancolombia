# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT cli.id_cliente, COUNT(cta.num_cuenta) AS cantidad_cuentas, SUM(cta.saldo) AS saldo_total
FROM Cliente cli
INNER JOIN Cuenta cta 
ON cli.id_cliente = cta.id_cliente
GROUP BY cli.id_cliente
HAVING COUNT(cta.num_cuenta) > 1
ORDER BY saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    cli.id_cliente,
    SUM(CASE WHEN trx.tipo_transaccion = 'deposito' THEN trx.monto END) AS total_depositos,
    SUM(CASE WHEN trx.tipo_transaccion = 'retiro' THEN trx.monto END) AS total_retiros
FROM Cliente cli
INNER JOIN Cuenta cta ON cli.id_cliente = cta.id_cliente
INNER JOIN Transaccion trx ON cta.num_cuenta = trx.num_cuenta
GROUP BY cli.id_cliente;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
--Realmente no hay manera ya que las cuentas no tienen tarjetas asociadas (Una cuenta puede existir sin tarjetas), sino las tarjetas tienen cuentas asociadas
-- adicionalmente el numero de cuenta es una llave foranea de la tabla tarjeta que no permite nulos, por tal motivo no habria
-- Pero si se necesitara revisar seria con el sigueinte query
SELECT 
    cta.num_cuenta,
    cta.tipo_cuenta
FROM Cuenta cta
INNER JOIN Tarjeta tarj ON cta.num_cuenta = tarj.num_cuenta
WHERE tarj.num_cuenta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    cta.tipo_cuenta,
    AVG(cta.saldo) AS saldo_promedio
FROM Cuenta cta
INNER JOIN Transaccion trx ON cta.num_cuenta = trx.num_cuenta
WHERE trx.fecha >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY cta.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT cli.id_cliente, cli.nombre
FROM Cliente cli
INNER JOIN Cuenta cta ON cli.id_cliente = cta.id_cliente
WHERE EXISTS (
    SELECT 1
    FROM Transaccion trx
    INNER JOIN Transferencia transf ON trx.id_transaccion = transf.id_transaccion
    WHERE trx.num_cuenta = cta.num_cuenta
)
AND NOT EXISTS (
    SELECT 1
    FROM Transaccion trx2
    INNER JOIN Retiro ret ON trx2.id_transaccion = ret.id_transaccion
    WHERE trx2.num_cuenta = cta.num_cuenta
      AND LOWER(ret.canal) = 'cajero'
);
```
