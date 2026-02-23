<p align="center">
<img src="assets/icon/logo-1024.png" alt="LYTOK Logo" width="250">
</p>

<p align="center">
<img src="https://img.shields.io/badge/version-2.0.0-blue" alt="Version">
<img src="https://img.shields.io/badge/license-Apache 2.0 | MIT-orange" alt="License">
<img src="https://img.shields.io/badge/format-.ltk-purple" alt="Format">
</p>

# LYTOK: Sintaxis de Alta Densidad(V2.0)

**LYTOK** (Lightweight Token Object Notation) es un estándar de serialización híbrido diseñado para maximizar la eficiencia en el intercambio de datos para Inteligencia Artificial y sistemas de alta concurrencia.

El formato utiliza la extensión oficial **.ltk** para textos planos y **.bltk** para binarios.

---

[!CAUTION]

⚠️ Major Update: v2.0 (Breaking Changes)

La versión 2.0 introduce cambios estructurales profundos. Los archivos .ltk creados con versiones anteriores no son compatibles con el nuevo parser universal. Se recomienda migrar a la nueva sintaxis de delimitación inteligente.

---

### 💡 ¿Por qué LYTOK?

En la era de los LLMs, cada carácter cuenta. JSON es excelente para humanos, pero ineficiente para máquinas:

- **Ahorro de Contexto:** Al eliminar la repetición de llaves en cada registro, LYTOK reduce drásticamente el consumo de tokens en la ventana de contexto.
- **Tipado Fuerte Nativo:** Diferencia entre enteros, flotantes, BigInts y fechas sin transformaciones costosas.
- **Optimización Estructural:** Su diseño permite a los motores de procesamiento implementar estrategias de alta velocidad (como Zero-Allocation) al reducir la ambigüedad y el ruido sintáctico.

### 🔄 ¿Qué cambió en la v2.0?

|     Característica     | v1.x (Legacy) |   v2.0 (Universal)    |
| :--------------------: | :-----------: | :-------------------: |
|  Separador de Campos   |  `\|` (Pipe)  |      `,` (Coma)       |
| Separador de Registros |    `$ / ;`    |  `;` (Punto y coma)   |
|         Header         | #Schema[N:]:  |     [...] o {...}     |
|        Strings         | \`Backticks\` |      "Comillas"       |
|       Versionado       | En el archivo | Universal (Agnóstico) |

### 🏗️ Modos de Operación

#### 1. Arreglos Uniformes (Colecciones).

Es el modo de máxima eficiencia. El esquema (Header) se define una vez al inicio.
**Ejemplo:** `[{id,#edad,activo?}]::` define la estructura para todos los registros siguientes.

#### 2. Arreglos Simples (#, ?, @, &, `)

Para listas de un solo tipo de dato primitivo.

- **Ejemplo (Números):** #1;2;3;4

- **Ejemplo (Fechas):** @"2025-01-01:00:00:00Z";"2025-02-01:00:00:00Z"

#### 3. Arreglos Mixtos (\*)

Para datos heterogéneos donde cada valor se auto-identifica.
**Ejemplo:** `\*#123,?t,un texto,^` (Donde ^ es Null).

---

### 💎 Tipos de Datos Avanzados

##### LYTOK soporta nativamente tipos que otros formatos de texto ignoran, garantizando cero pérdida de precisión:

| Símbolo |  Tipo   |                                     Regla Técnica                                     |
| :-----: | :-----: | :-----------------------------------------------------------------------------------: |
|    #    | Number  |         Soporta Integers y Floats detectados por la presencia del punto `.`.          |
|    &    | BigInt  |                          Soporta enteros de hasta 128 bits.                           |
|    ?    | Boolean |                     Representado por `?t` (True) o `?f` (False).                      |
|    @    |  Date   |      Obligatorio formato ISO 8601 (YYYY-MM-DDTHH:mm:ss.sssZ) para parseo nativo.      |
|    `    |  Text   | Encapsulamiento con comillas. Escapes internos como `""` y saltos de línea como `\n.` |
|    ^    |  Null   |                           Representa la ausencia de valor.                            |
|   \*    |  Mixed  |          Permite que el campo acepte cualquier tipo soportado dinámicamente.          |

### 📏 Reglas de Estructura y autodelimitación

**1. Separadores Estructurales:**

- **`,`:** Separador de campos (opcional si hay autodelimitación).
- **`;`:** Separador de registros o elementos en arreglos.
- **`::`:** Delimitador que separa el Esquema (Header) de la Data.

**2. Autodelimitación (Ahorro de Bytes):** LYTOK permite omitir la coma , cuando el cambio de tipo es inequívoco:

- **Strings ("..."):** El cierre de comillas actúa como delimitador.
- **Estructuras ({} []):** Las llaves y corchetes marcan el inicio/fin de bloques.
- **Nulidad (^):** El motor identifica el nulo instantáneamente.

**3. Formato y Escapes:**

- Los strings complejos usan `"`. Si un string contiene una comilla, se escapa duplicándola: `"Dijo ""Hola"""`.
- Los saltos de línea se representan con el escape literal `\n`.

#### Ejemplo Complejo (V2.0):

```Text
[
    id,fecha,#stock_total,?es_importado,?es_certificado,fabricante{
        nombre,*registro,direccion{
            calle,ciudad,#zip
        }
    },datosLaboratorio{
        fecha_prueba,direccion{
            calle,edificio,referencia
        }
    },certificaciones[
        nombre_cert,valida_hasta
    ],sucursales[
        nombre,ciudad,empleados[
            nombre,apellido,datos{
                calle,#numero,#telefono,correo
            }
        ]
    ]]::
L|001,2025-11-20,50000,f,t{
    Acme Corp,0123456789{
        Calle Falsa 123,Springfield,62000
    }
}^[
    ISO-9001,2026-01-01;
    CQC-A,2027-05-15
][
    suc-52,La Paz[
        Maria,Perez{
            calle imaginaria,520,6489635672,maria.perez@mail.com
        };
        "Eusebio;"Mendez{
            calle imaginaria sur,580,6489635652,eusebio.m@mail.com
        }
    ];
    suc-105,Tepito^
];
L002,2025-11-20,15000,t,f{
    Industrias Z,#987654321{
        Av. Siempre Viva 742,CDMX,90210
    }
}{
    2025-11-25{
        Rio Tiber,Torre Central,2ndo piso
    }
}[
    ISO-9001,2026-01-01;
    Cert_MX_IM,2026-01-01
]^
```

### 📂 Estructura del Proyecto

- `/spec`: Gramática formal EBNF V2.0.

- `/compliance`: El Test Suite oficial para validacion de parsers.

- `/assets`: Identidad visual y logotipos.

### SDK's disponibles

- #### [JS/TS](https://github.com/Joguel96/lytok-js)

### Prueba el laboratorio gratuito

[lytok lab](https://lytoklab.netlify.app/)

---

### 📜 Licencia

- #### Especificación (Gramática y Reglas): [Apache License 2.0](https://github.com/Joguel96/lytok-spec/blob/main/spec/LICENSE)

- #### Compliance Suite y Herramientas: [MIT License](https://github.com/Joguel96/lytok-spec/blob/main/compliance/LICENSE)

---

_Desarrollado por [joguel96](https://github.com/joguel96). LYTOK es la respuesta a la necesidad de un transporte de datos más inteligente y ligero._
