# Fix para `Parse error: unexpected token ".", expecting "," or ";"`

En Scriptcase (Blank app), este error en PHP 8.1 suele aparecer cuando una expresión de concatenación queda partida dentro de una macro SQL y el parser generado por Scriptcase rompe la sentencia.

## Ajuste recomendado

Antes de las consultas `sc_lookup`, armá los valores en variables simples y usalas en el SQL sin concatenaciones encadenadas largas.

### Ejemplo seguro para el bloque de usuario por `id`

```php
$jm_uid = (int)$jm_session_user_id;
$jm_sql_user = "
    SELECT
        u.id,
        u.username,
        CONCAT(TRIM(COALESCE(u.nombre,'')), ' ', TRIM(COALESCE(u.apellido,''))) AS full_name,
        COALESCE(u.email,'') AS email,
        COALESCE(GROUP_CONCAT(DISTINCT r.codigo ORDER BY r.id SEPARATOR ','), '') AS roles_csv,
        COALESCE(GROUP_CONCAT(DISTINCT r.nombre ORDER BY r.id SEPARATOR ', '), '') AS roles_names
    FROM alq_usuarios u
    LEFT JOIN alq_usuarios_roles ur
        ON ur.usuario_id = u.id
        AND ur.activo = 1
    LEFT JOIN alq_roles r
        ON r.id = ur.rol_id
        AND r.activo = 1
    WHERE u.id = {$jm_uid}
        AND u.activo = 1
        AND u.eliminado = 0
    GROUP BY u.id
    LIMIT 1
";

sc_lookup(jm_rs_user, $jm_sql_user);
```

### Ejemplo seguro para el bloque por `username`

```php
$jm_username_sql = jm_sql($jm_session_username);
$jm_sql_user2 = "
    SELECT
        u.id,
        u.username,
        CONCAT(TRIM(COALESCE(u.nombre,'')), ' ', TRIM(COALESCE(u.apellido,''))) AS full_name,
        COALESCE(u.email,'') AS email,
        COALESCE(GROUP_CONCAT(DISTINCT r.codigo ORDER BY r.id SEPARATOR ','), '') AS roles_csv,
        COALESCE(GROUP_CONCAT(DISTINCT r.nombre ORDER BY r.id SEPARATOR ', '), '') AS roles_names
    FROM alq_usuarios u
    LEFT JOIN alq_usuarios_roles ur
        ON ur.usuario_id = u.id
        AND ur.activo = 1
    LEFT JOIN alq_roles r
        ON r.id = ur.rol_id
        AND r.activo = 1
    WHERE u.username = '{$jm_username_sql}'
        AND u.activo = 1
        AND u.eliminado = 0
    GROUP BY u.id
    LIMIT 1
";

sc_lookup(jm_rs_user2, $jm_sql_user2);
```

## Checklist rápido

1. Volvé a pegar el código en el evento `onExecute` del Blank (no en un campo de texto con `\n` literales).
2. Confirmá que no haya texto extra después del cierre del heredoc final (`HTML;`).
3. Regenerá la aplicación (`Run > Generate Source`).
4. Si persiste, abrí el `index.php` generado y revisá exactamente la línea reportada para ver qué sentencia quedó truncada.
