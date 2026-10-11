# Git Flow

Se tarbajará con **Git Flow clásico**, donde `develop` **es precisamente la rama donde se integran las features**. Esa es la idea central del modelo.

## 1. Las 5 ramas

```text
main
develop
feature/*
release/*
hotfix/*
```

Su relación es:

```text
                         feature/A
                        /
                       /
main ─────────────── develop
  │                  /   \
  │                 /     feature/B
  │                /
  │               release/1.0
  │                    │
  │                    ▼
  └────────────────── main
                       │
                       ▼
                    v1.0.0
```

Detalle importantísimo:

> **`develop` representa el estado acumulado de desarrollo, no necesariamente algo listo para producción.**

---

# 2. `main`

`main` contiene exclusivamente código que consideras **production-ready**.

```text
main

A──B──C──D
         ↑
       v1.2.0
```

Cada commit importante en `main` debería corresponder a una versión liberada a producción.

Por ejemplo:

```text
v1.0.0
v1.1.0
v1.2.0
```

No se tranaja directamente ahí.

---

# 3. `develop`

`develop` es la **rama principal de integración**.

Parte inicialmente de `main`:

```text
main
  │
  └──── develop
```

Pero después empieza a avanzar independientemente:

```text
main     A────B──────────────C
              \
develop        B──D──E──F────G
```

Aquí `D`, `E`, `F`, `G` representan cambios que todavía no están en producción.

Por eso:

```text
main    = producción
develop = próximo desarrollo
```

---

# 4. `feature/*`

Una feature **siempre nace de `develop`**.

Supongamos:

```text
develop
   │
   └── feature/login
```

Trabajas:

```text
develop
   │
   └── feature/login
          │
          A
          B
          C
```

Mientras trabajas en ella, **`develop` puede continuar avanzando** con otras features.

Por ejemplo:

```text
                    feature/login
                   /
develop ──────────A────────B────C
         \
          \
           feature/payment
                │
                D
                E
```

Cada feature está aislada.

---

# 5. ¿Cuándo entra la feature a `develop`?

Cuando está terminada.

```text
feature/login
       │
       │ merge
       ▼
develop
```

Queda:

```text
develop

A──B──C────M
          ↑
     login integrado
```

En esta parte hay que cuidar que no se "contamine" develop con features incompletas

Si tienes:

```text
feature/A
feature/B
feature/C
```

y haces:

```text
feature/A ──► develop
feature/B ──► develop
feature/C ──► develop
```

`develop` contiene las tres.

Eso **no es un error de Git Flow clásico**.

Es justamente lo que el modelo pretende.

El supuesto es que:

> **una feature que entra a `develop` ya está suficientemente terminada como para formar parte de la próxima versión.**

---

# 6. Entonces, ¿qué pasa si B no debería salir todavía?

Aquí está la parte que suele causar confusión.

Git Flow clásico **no intenta solucionar esto con branches**.

El modelo asume que si `feature/B` no debe formar parte de la próxima release:

**no se mergea a `develop`.**

Por ejemplo:

```text
develop
   │
   ├── feature/A ✅
   │
   ├── feature/B 🚧
   │
   └── feature/C 🚧
```

Solo haces:

```text
feature/A → develop
```

Y mantienes:

```text
feature/B
feature/C
```

fuera.

---

# 7. ¿Y si A depende de B?

Aquí empieza a ponerse interesante.

Supongamos:

```text
feature/A
     ↓
necesita
     ↓
feature/B
```

No deberías hacer:

```text
feature/B → develop
feature/A → develop
```

si B todavía no está lista.

Puedes hacer:

```text
develop
   │
   └── feature/B
            │
            └── feature/A
```

Es decir:

```bash
git checkout feature/B
git checkout -b feature/A
```

Entonces:

```text
develop
   │
   └── feature/B
          │
          └── feature/A
```

Cuando B esté lista:

```text
feature/B → develop
```

Y posteriormente A.

---

# 8. `release/*`

Cuando `develop` tiene **exactamente el conjunto de funcionalidades que quieres liberar**, se crea:

```text
release/1.4.0
```

desde `develop`.

```text
develop
   │
   └── release/1.4.0
```

Ahora la release queda congelada en cuanto a features.

```text
develop
   │
   ├── feature/X
   │
   └── release/1.4.0
```

Mientras la release está siendo probada:

```text
release/1.4.0
       │
       ├── bug fix
       ├── bug fix
       └── config
```

Pero **no agregas nuevas features**.

---

# 9. ¿Qué pasa con `develop` mientras existe una release?

Puede continuar.

```text
                       feature/X
                      /
develop ─────────────M────N────O
                    /
                   /
release/1.4.0 ────R────R────R
```

Esto es una de las grandes ventajas de Git Flow.

Se puede estar haciendo:

```text
release/1.4.0
```

al mismo tiempo que se empieza:

```text
feature/1.5
```

---

# 10. La release termina en `main`

Cuando `release/1.4.0` está lista:

```text
release/1.4.0
       │
       ├──────────► main
       │
       └──────────► develop
```

Se hace merge a `main`:

```text
main
 A──B──C────R
            ↑
          v1.4.0
```

Y se crea el tag:

```bash
git tag v1.4.0
```

---

# 11. ¿Por qué también vuelve a `develop`?

Porque durante la release pudieron existir correcciones.

Ejemplo:

```text
develop
     │
     └── release/1.4.0
              │
              ├── fix/A
              ├── fix/B
              └── fix/C
```

Esas correcciones tienen que regresar a `develop`.

Por eso:

```text
release/1.4.0 → main
release/1.4.0 → develop
```

Así no se pierden los fixes.

---

# 12. `hotfix/*`

Ahora imagina que producción tiene un problema crítico.

```text
main
   │
   └── v1.4.0
          │
          └── ERROR
```

No se requiere esperar a que termine `develop`.

Entonces:

```text
main
  │
  └── hotfix/1.4.1
```

Corrige:

```text
hotfix/1.4.1
       │
       └── fix
```

Y después:

```text
hotfix/1.4.1
       │
       ├────────► main
       │
       └────────► develop
```

Produce:

```text
v1.4.1
```

---

# 13. El flujo completo clásico

Visualmente:

```text
                         feature/A
                        /        \
                       /          \
                      /            \
                     /              \
main ────────────────┼───────────────┼──────────────
  │                  │               │
  │                  ▼               │
  │               develop            │
  │                  │               │
  │                  │               │
  │                  ▼               │
  │             release/1.0          │
  │                  │               │
  │                  ├──────────────►│
  │                  │               │
  │                  ▼               │
  │                main              │
  │                  │               │
  │                 v1.0              │
  │                                  │
  └── hotfix/1.0.1 ──────────────────┘
```

Simplificado:

```text
              feature/*
             /
            /
develop ───┼──────► release/* ───► main
            \                         │
             \                        │
              feature/*              │
                                      │
                       hotfix/* ◄─────┘
```

---

# 14. Las reglas del Git Flow clásico



### `main`

```text
Producción
```

Nunca se desarrolla directamente ahí.

### `develop`

```text
Integración de la próxima versión
```

Recibe features terminadas.

### `feature/*`

```text
Nueva funcionalidad
```

Nace de:

```text
develop
```

Termina en:

```text
develop
```

### `release/*`

```text
Preparación de una versión
```

Nace de:

```text
develop
```

Termina en:

```text
main + develop
```

### `hotfix/*`

```text
Corrección urgente de producción
```

Nace de:

```text
main
```

Termina en:

```text
main + develop
```

---

## Prevención de "develop contaminado"

Aquí está la distinción importante:

**Si se está usando Git Flow clásico correctamente, `develop` puede contener múltiples features. Eso no se considera contaminación.**

La contaminación ocurre cuando:

```text
feature incompleta
        ↓
     develop
        ↓
   release
        ↓
      main
```

La solución clásica es:

```text
feature incompleta
        ↓
   NO entra a develop
```

Mientras que:

```text
feature terminada
        ↓
     develop
        ↓
   release
        ↓
      main
```

Es por eso, que se requiere usar **Git Flow clásico puro**, y mantener exactamente estas cinco categorías:

```text
main
develop
feature/*
release/*
hotfix/*
```

y la regla crítica es:

> **Una feature solamente se integra a `develop` cuando está terminada y es candidata a formar parte de la próxima release.**

