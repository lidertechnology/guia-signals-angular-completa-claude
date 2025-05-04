# Signals en Angular: Guía Completa

## Introducción

Los Signals son una característica introducida en Angular 16 que representan una nueva forma de gestionar el estado y la reactividad en aplicaciones Angular. Funcionan como contenedores de valores que notifican a los consumidores cuando estos valores cambian, creando un sistema de reactividad granular y eficiente.

## ¿Qué son los Signals?

Un Signal es simplemente un contenedor de un valor que notifica a los consumidores interesados cuando ese valor cambia. Técnicamente, es una función que:

1. Devuelve su valor actual cuando se invoca
2. Puede ser actualizada usando métodos específicos
3. Notifica automáticamente a todos los consumidores cuando cambia

## Ejemplos Básicos

### Creación de un Signal Básico

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-contador',
  template: `
    <h2>Contador: {{ contador() }}</h2>
    <button (click)="incrementar()">Incrementar</button>
    <button (click)="decrementar()">Decrementar</button>
  `
})
export class ContadorComponent {
  // Crear un signal con valor inicial 0
  contador = signal(0);
  
  incrementar() {
    // Actualizar el valor del signal
    this.contador.update(valor => valor + 1);
  }
  
  decrementar() {
    this.contador.update(valor => valor - 1);
  }
}
```

En este ejemplo simple:
- Creamos un signal `contador` con valor inicial 0
- Para leer el valor, invocamos el signal como función: `contador()`
- Para actualizar el valor, usamos el método `update`

### Otras formas de modificar un Signal

```typescript
// Establecer un valor directamente
this.contador.set(10);

// Actualizar basado en el valor actual
this.contador.update(valor => valor * 2);
```

## Computed Signals

Los Computed Signals son signals que se derivan de otros signals. Se recalculan automáticamente cuando cualquiera de sus signals dependientes cambia.

### Ejemplo de Computed Signal

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-carrito',
  template: `
    <div>
      <h3>Carrito de Compras</h3>
      <p>Cantidad: {{ cantidad() }}</p>
      <p>Precio por unidad: {{ precioPorUnidad() }}€</p>
      <p>Total: {{ precioTotal() }}€</p>
      
      <button (click)="agregarProducto()">Agregar</button>
      <button (click)="quitarProducto()">Quitar</button>
    </div>
  `
})
export class CarritoComponent {
  cantidad = signal(1);
  precioPorUnidad = signal(25);
  
  // Computed signal que depende de otros signals
  precioTotal = computed(() => this.cantidad() * this.precioPorUnidad());
  
  agregarProducto() {
    this.cantidad.update(valor => valor + 1);
  }
  
  quitarProducto() {
    if (this.cantidad() > 0) {
      this.cantidad.update(valor => valor - 1);
    }
  }
}
```

En este ejemplo:
- Tenemos dos signals básicos: `cantidad` y `precioPorUnidad`
- Creamos un computed signal `precioTotal` que se calcula a partir de los otros dos
- Cuando cambia cualquier signal dependiente, `precioTotal` se recalcula automáticamente

## Casos de Uso de los Signals

### 1. Gestión de Estado Local de Componentes

Los signals son perfectos para manejar el estado interno de un componente:

```typescript
@Component({
  selector: 'app-formulario',
  template: `
    <div>
      <input 
        [value]="nombre()" 
        (input)="actualizarNombre($event)"
        placeholder="Nombre"
      />
      <p *ngIf="nombreEsValido()">✅ Nombre válido</p>
      <p *ngIf="!nombreEsValido()">❌ El nombre debe tener al menos 3 caracteres</p>
    </div>
  `
})
export class FormularioComponent {
  nombre = signal('');
  nombreEsValido = computed(() => this.nombre().length >= 3);
  
  actualizarNombre(event: Event) {
    const input = event.target as HTMLInputElement;
    this.nombre.set(input.value);
  }
}
```

### 2. Formularios Reactivos

```typescript
@Component({
  selector: 'app-registro',
  template: `
    <form>
      <div>
        <label>Correo:</label>
        <input [value]="email()" (input)="actualizarEmail($event)" />
        <span *ngIf="!emailEsValido()">Correo inválido</span>
      </div>
      
      <div>
        <label>Contraseña:</label>
        <input type="password" [value]="password()" (input)="actualizarPassword($event)" />
        <span *ngIf="!passwordEsValida()">La contraseña debe tener al menos 8 caracteres</span>
      </div>
      
      <button [disabled]="!formularioEsValido()">Registrarse</button>
    </form>
  `
})
export class RegistroComponent {
  email = signal('');
  password = signal('');
  
  emailEsValido = computed(() => {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(this.email());
  });
  
  passwordEsValida = computed(() => this.password().length >= 8);
  
  formularioEsValido = computed(() => 
    this.emailEsValido() && this.passwordEsValida()
  );
  
  actualizarEmail(event: Event) {
    const input = event.target as HTMLInputElement;
    this.email.set(input.value);
  }
  
  actualizarPassword(event: Event) {
    const input = event.target as HTMLInputElement;
    this.password.set(input.value);
  }
}
```

### 3. Comunicación entre Componentes

```typescript
// Servicio compartido
@Injectable({
  providedIn: 'root'
})
export class UsuarioService {
  // Estado global de la aplicación
  nombreUsuario = signal('');
  estaAutenticado = signal(false);
  
  iniciarSesion(nombre: string) {
    this.nombreUsuario.set(nombre);
    this.estaAutenticado.set(true);
  }
  
  cerrarSesion() {
    this.nombreUsuario.set('');
    this.estaAutenticado.set(false);
  }
}

// Componente de formulario de login
@Component({
  selector: 'app-login',
  template: `
    <div>
      <input [value]="nombre()" (input)="actualizarNombre($event)" placeholder="Nombre" />
      <button (click)="login()">Iniciar sesión</button>
    </div>
  `
})
export class LoginComponent {
  nombre = signal('');
  
  constructor(private usuarioService: UsuarioService) {}
  
  actualizarNombre(event: Event) {
    const input = event.target as HTMLInputElement;
    this.nombre.set(input.value);
  }
  
  login() {
    this.usuarioService.iniciarSesion(this.nombre());
  }
}

// Componente de cabecera que muestra el estado de autenticación
@Component({
  selector: 'app-cabecera',
  template: `
    <header>
      <ng-container *ngIf="estaAutenticado()">
        Hola, {{ nombreUsuario() }}
        <button (click)="logout()">Cerrar sesión</button>
      </ng-container>
      <ng-container *ngIf="!estaAutenticado()">
        No has iniciado sesión
      </ng-container>
    </header>
  `
})
export class CabeceraComponent {
  constructor(private usuarioService: UsuarioService) {}
  
  // Exponer los signals del servicio para usarlos en la plantilla
  estaAutenticado = this.usuarioService.estaAutenticado;
  nombreUsuario = this.usuarioService.nombreUsuario;
  
  logout() {
    this.usuarioService.cerrarSesion();
  }
}
```

## Ventajas de los Signals en Aplicaciones Modernas

### 1. Rendimiento Mejorado

Los Signals ofrecen una reactividad granular y eficiente:

- **Detección de cambios localizada**: Solo se actualizan los componentes que realmente dependen de un valor cambiado.
- **Menos ciclos de detección de cambios**: Evita verificaciones innecesarias en toda la aplicación.
- **Actualizaciones eficientes del DOM**: Solo se renderizan las partes de la vista que realmente han cambiado.

### 2. Mejor Mantenibilidad del Código

- **Transparencia en las dependencias**: Es más fácil entender qué afecta a qué en tu aplicación.
- **Código declarativo**: La lógica de transformación de datos se declara una vez y se actualiza automáticamente.
- **Separación clara de lectura y escritura**: Los signals distinguen claramente entre obtener valores y actualizarlos.

### 3. Integración con el Resto del Ecosistema Angular

- **Compatibilidad con Angular existente**: Puedes adoptarlos gradualmente en aplicaciones existentes.
- **Funcionan con otras APIs de Angular**: Se integran bien con pipes, directivas y otros componentes.
- **Preparados para el futuro**: Son parte de la visión a largo plazo para la reactividad en Angular.

### 4. Depuración más Sencilla

- **Trazabilidad de cambios**: Es más fácil rastrear de dónde vienen los cambios.
- **Menos efectos secundarios ocultos**: Los flujos de datos son más predecibles.
- **Código más fácil de probar**: Los signals facilitan la creación de pruebas unitarias.

## Interacción entre Componentes mediante Signals

### 1. Comunicación Padre-Hijo

#### Componente Padre
```typescript
@Component({
  selector: 'app-padre',
  template: `
    <div>
      <h2>Componente Padre</h2>
      <p>Contador: {{ contador() }}</p>
      <button (click)="incrementar()">+</button>
      <button (click)="decrementar()">-</button>
      
      <!-- Pasamos el signal al hijo -->
      <app-hijo [valorContador]="contador" (incrementar)="incrementar()"></app-hijo>
    </div>
  `
})
export class PadreComponent {
  contador = signal(0);
  
  incrementar() {
    this.contador.update(val => val + 1);
  }
  
  decrementar() {
    this.contador.update(val => val - 1);
  }
}
```

#### Componente Hijo
```typescript
@Component({
  selector: 'app-hijo',
  template: `
    <div>
      <h3>Componente Hijo</h3>
      <p>Valor del padre: {{ valorContador() }}</p>
      <p>Valor duplicado: {{ valorDuplicado() }}</p>
      <button (click)="onIncrementar()">Incrementar desde hijo</button>
    </div>
  `
})
export class HijoComponent {
  // Recibimos el signal desde el padre
  @Input() valorContador!: Signal<number>;
  @Output() incrementar = new EventEmitter<void>();
  
  // Creamos un computed signal basado en el signal del padre
  valorDuplicado = computed(() => this.valorContador() * 2);
  
  onIncrementar() {
    this.incrementar.emit();
  }
}
```

### 2. Patrón de Servicio Compartido

Un enfoque más escalable es utilizar un servicio para compartir signals entre componentes no relacionados:

#### Servicio de Estado Compartido
```typescript
@Injectable({
  providedIn: 'root'
})
export class EstadoAppService {
  // Estado de la aplicación
  contador = signal(0);
  usuario = signal<Usuario | null>(null);
  
  // Computed signals
  tieneUsuario = computed(() => this.usuario() !== null);
  contadorPar = computed(() => this.contador() % 2 === 0);
  
  incrementarContador() {
    this.contador.update(val => val + 1);
  }
  
  establecerUsuario(usuario: Usuario) {
    this.usuario.set(usuario);
  }
  
  cerrarSesion() {
    this.usuario.set(null);
  }
}
```

#### Componentes consumidores
```typescript
@Component({
  selector: 'app-contador-widget',
  template: `
    <div>
      <p>Contador: {{ estado.contador() }}</p>
      <p *ngIf="estado.contadorPar()">El contador es par</p>
      <button (click)="estado.incrementarContador()">Incrementar</button>
    </div>
  `
})
export class ContadorWidgetComponent {
  constructor(public estado: EstadoAppService) {}
}

@Component({
  selector: 'app-perfil-usuario',
  template: `
    <div *ngIf="estado.tieneUsuario()">
      <h3>Perfil de {{ estado.usuario()?.nombre }}</h3>
      <p>Email: {{ estado.usuario()?.email }}</p>
      <button (click)="estado.cerrarSesion()">Cerrar sesión</button>
    </div>
    <div *ngIf="!estado.tieneUsuario()">
      No has iniciado sesión
    </div>
  `
})
export class PerfilUsuarioComponent {
  constructor(public estado: EstadoAppService) {}
}
```

### 3. Usando `effect()` para Reaccionar a Cambios

El método `effect()` permite ejecutar código cuando los signals cambian:

```typescript
@Component({
  selector: 'app-notificaciones',
  template: `
    <div>
      <h3>Notificaciones</h3>
      <p>Mensajes no leídos: {{ mensajesNoLeidos() }}</p>
    </div>
  `
})
export class NotificacionesComponent implements OnInit {
  mensajesNoLeidos = signal(0);
  
  constructor(private notificacionService: NotificacionService) {}
  
  ngOnInit() {
    // Efecto que se ejecuta cuando cambian los mensajes no leídos
    effect(() => {
      const cantidad = this.mensajesNoLeidos();
      if (cantidad > 0) {
        // Actualizar el título de la página
        document.title = `(${cantidad}) Nuevos mensajes`;
      } else {
        document.title = 'Mi Aplicación';
      }
    });
    
    // Simular recepción de mensajes
    setTimeout(() => this.mensajesNoLeidos.set(3), 3000);
  }
}
```

## Patrones Avanzados con Signals

### 1. Sincronización con APIs Externas

```typescript
@Injectable({
  providedIn: 'root'
})
export class ProductosService {
  productos = signal<Producto[]>([]);
  cargando = signal(false);
  error = signal<string | null>(null);
  
  cargarProductos() {
    this.cargando.set(true);
    this.error.set(null);
    
    this.http.get<Producto[]>('/api/productos').pipe(
      finalize(() => this.cargando.set(false))
    ).subscribe({
      next: (data) => this.productos.set(data),
      error: (err) => this.error.set(err.message)
    });
  }
}

@Component({
  selector: 'app-lista-productos',
  template: `
    <div>
      <h2>Productos</h2>
      <p *ngIf="productosService.cargando()">Cargando...</p>
      <p *ngIf="productosService.error()">Error: {{ productosService.error() }}</p>
      
      <ul *ngIf="!productosService.cargando() && !productosService.error()">
        <li *ngFor="let producto of productosService.productos()">
          {{ producto.nombre }} - {{ producto.precio }}€
        </li>
      </ul>
      
      <button (click)="productosService.cargarProductos()">Recargar</button>
    </div>
  `
})
export class ListaProductosComponent implements OnInit {
  constructor(public productosService: ProductosService) {}
  
  ngOnInit() {
    this.productosService.cargarProductos();
  }
}
```

### 2. Signals para Filtrado y Ordenamiento

```typescript
@Component({
  selector: 'app-tabla-datos',
  template: `
    <div>
      <input placeholder="Filtrar..." [value]="filtroTexto()" 
             (input)="actualizarFiltro($event)">
      
      <select [value]="ordenarPor()" (change)="cambiarOrden($event)">
        <option value="nombre">Nombre</option>
        <option value="precio">Precio</option>
        <option value="fecha">Fecha</option>
      </select>
      
      <table>
        <thead>
          <tr>
            <th>Nombre</th>
            <th>Precio</th>
            <th>Fecha</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let item of elementosFiltrados()">
            <td>{{ item.nombre }}</td>
            <td>{{ item.precio }}€</td>
            <td>{{ item.fecha | date }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  `
})
export class TablaDatosComponent {
  // Datos originales
  elementos = signal<Elemento[]>([
    { nombre: 'Producto A', precio: 25, fecha: new Date(2023, 5, 10) },
    { nombre: 'Servicio B', precio: 50, fecha: new Date(2023, 4, 15) },
    { nombre: 'Artículo C', precio: 30, fecha: new Date(2023, 6, 5) }
  ]);
  
  // Filtros y orden
  filtroTexto = signal('');
  ordenarPor = signal('nombre');
  
  // Computed signal para elementos filtrados y ordenados
  elementosFiltrados = computed(() => {
    let resultado = [...this.elementos()];
    
    // Aplicar filtro
    const filtro = this.filtroTexto().toLowerCase();
    if (filtro) {
      resultado = resultado.filter(item => 
        item.nombre.toLowerCase().includes(filtro)
      );
    }
    
    // Aplicar orden
    const campo = this.ordenarPor();
    resultado.sort((a, b) => {
      if (a[campo] < b[campo]) return -1;
      if (a[campo] > b[campo]) return 1;
      return 0;
    });
    
    return resultado;
  });
  
  actualizarFiltro(event: Event) {
    const input = event.target as HTMLInputElement;
    this.filtroTexto.set(input.value);
  }
  
  cambiarOrden(event: Event) {
    const select = event.target as HTMLSelectElement;
    this.ordenarPor.set(select.value);
  }
}
```

## Conclusión

Los Signals en Angular representan un avance significativo en la forma en que gestionamos el estado y la reactividad en aplicaciones Angular modernas. Sus ventajas principales son:

1. **Rendimiento optimizado** con detección de cambios granular
2. **Mantenibilidad mejorada** con transparencia en las dependencias
3. **Código más declarativo** con transformaciones automáticas de datos
4. **Mejor integración** con el ecosistema Angular
5. **Depuración simplificada** con flujos de datos más predecibles

La capacidad de los Signals para facilitar la comunicación entre componentes, ya sea mediante la compartición directa de signals o a través de servicios, hace que sean una herramienta poderosa para aplicaciones Angular modernas y escalables.

A medida que Angular continúa evolucionando, los Signals se están convirtiendo en un elemento central del framework, proporcionando un enfoque coherente y eficiente para la gestión del estado reactivo.
