# XRCX_Puzzlebot

Stack de navegación autónoma en ROS 2 para el robot Puzzlebot, con soporte para SLAM, simulación en Gazebo y navegación con Nav2.

## Integrantes

| Nombre | GitHub |
| --- | --- |
| Sofía Blanco Prigmore | `AifosWhite` |
| Josué Aldemar Garduño Gómez | `aldemar3002` |
| Karina Fernanda Maldonado Murillo | `thephoeniix` |
| Roberto Carlos Pedraza Miranda | `RoberttCap` |

## Descripción general

Este repositorio contiene el stack de navegación en ROS 2 para el robot Puzzlebot, estructurado en tres paquetes con responsabilidades claramente separadas:

- `puzzlebot_description` define cómo es el robot: modelo, URDF/Xacro, mallas y frames.
- `puzzlebot_gazebo` define cómo se ejecuta el robot en simulación: mundo, bridge ROS-Gazebo y spawn.
- `puzzlebot_navigation` define cómo se comporta el stack de SLAM y navegación: `slam_toolbox`, Nav2, RViz y mapas.

Nota: aunque a veces se le mencione como `puzzlebot_navigation2`, en este repositorio el nombre real del paquete es `puzzlebot_navigation`.

## Estructura del repositorio

```text
puzzlebot_ros2/
├── puzzlebot_description/
│   ├── launch/
│   ├── meshes/
│   ├── rviz/
│   └── urdf/
├── puzzlebot_gazebo/
│   ├── config/
│   ├── launch/
│   └── worlds/
└── puzzlebot_navigation/
    ├── config/
    ├── launch/
    ├── maps/
    └── rviz/
```

## Requisitos

Antes de correr el proyecto, asegúrate de tener instalado:

- ROS 2 con `colcon`
- `xacro`
- `robot_state_publisher`
- `rviz2`
- `joint_state_publisher_gui`
- `ros_gz_sim`
- `ros_gz_bridge`
- `slam_toolbox`
- `nav2_bringup`
- `teleop_twist_keyboard`
- `xterm`

También necesitas tener este repositorio dentro de la carpeta `src` de un workspace de ROS 2.

## Compilación

Desde la raíz del workspace, por ejemplo `~/puzzlebot_ws`, ejecuta:

```bash
rosdep install --from-paths src --ignore-src -r -y
colcon build --packages-select puzzlebot_description puzzlebot_gazebo puzzlebot_navigation
source install/setup.bash
```

Si ya compilaste antes y abriste una nueva terminal, recuerda volver a hacer:

```bash
source ~/puzzlebot_ws/install/setup.bash
```

## Paquetes

### `puzzlebot_description`

Este paquete contiene la descripción del robot.

Incluye:

- `urdf/puzzlebot.xacro`: modelo principal del Puzzlebot.
- `meshes/`: mallas STL de la base y ruedas.
- `rviz/puzzlebot_description.rviz`: configuración de RViz para visualizar el robot.
- `launch/puzzlebot_description.launch.xml`: lanza `robot_state_publisher` y, opcionalmente, RViz, Gazebo y la GUI de joints.

Comando base:

```bash
ros2 launch puzzlebot_description puzzlebot_description.launch.xml rviz:=true
```

Argumentos útiles:

- `rviz:=true` abre RViz con la configuración del robot.
- `joint_gui:=true` abre `joint_state_publisher_gui`.
- `gazebo:=true` levanta una simulación vacía y spawnea el robot.
- `use_sim_time:=true` usa el reloj de simulación.

Ejemplo:

```bash
ros2 launch puzzlebot_description puzzlebot_description.launch.xml rviz:=true joint_gui:=true
```

### `puzzlebot_gazebo`

Este paquete contiene la simulación del robot en Gazebo.

Incluye:

- `worlds/maze.world`: mundo de simulación.
- `config/gazebo_bridge.yaml`: bridge entre tópicos de Gazebo y ROS 2.
- `launch/puzzlebot_gazebo.launch.xml`: abre el mundo, publica la descripción del robot, lo inserta en Gazebo y activa el bridge.

Este launch:

- carga el mundo `maze.world`
- spawnea al robot en la pose inicial
- publica `/clock`, `/odom`, `/tf`, `/joint_states` y `/scan`
- recibe `/cmd_vel` desde ROS 2 para mover el robot en la simulación

Comando base:

```bash
ros2 launch puzzlebot_gazebo puzzlebot_gazebo.launch.xml
```

Argumentos útiles:

- `headless:=false` abre la interfaz gráfica de Gazebo.
- `headless:=true` corre Gazebo sin interfaz.
- `use_sim_time:=true` usa tiempo de simulación.

Ejemplo:

```bash
ros2 launch puzzlebot_gazebo puzzlebot_gazebo.launch.xml headless:=false
```

### `puzzlebot_navigation`

Este paquete contiene la parte de percepción y navegación autónoma.

Incluye:

- `config/slam_toolbox.yaml`: parámetros de SLAM.
- `config/nav2_params.yaml`: parámetros de Nav2 y AMCL.
- `maps/map.yaml` y `maps/map.pgm`: mapa por defecto para localización y navegación.
- `rviz/slam.rviz`: configuración de RViz para mapeo.
- `rviz/nav2.rviz`: configuración de RViz para navegación.
- `launch/slam_core.launch.xml`: corre `slam_toolbox`, RViz y teleoperación por teclado.
- `launch/slam.launch.xml`: levanta Gazebo y luego el flujo de SLAM.
- `launch/nav2_core.launch.xml`: levanta Nav2 con un mapa y RViz.
- `launch/nav2.launch.xml`: levanta Gazebo y luego Nav2.

## Cómo correr el proyecto

### 1. Visualizar solo el robot

Útil para verificar que el modelo, frames y mallas cargan correctamente.

```bash
ros2 launch puzzlebot_description puzzlebot_description.launch.xml rviz:=true
```

### 2. Correr solo la simulación en Gazebo

Útil para validar el mundo, sensores y bridge sin levantar navegación.

```bash
ros2 launch puzzlebot_gazebo puzzlebot_gazebo.launch.xml headless:=false
```

### 3. Ejecutar SLAM en simulación

Este flujo levanta:

- Gazebo con el mundo `maze.world`
- el robot Puzzlebot
- `slam_toolbox`
- RViz para mapeo
- teleoperación por teclado

Comando:

```bash
ros2 launch puzzlebot_navigation slam.launch.xml headless:=false
```

Notas:

- El launch abre `teleop_twist_keyboard` usando `xterm -e`.
- Usa `use_sim_time:=true` por defecto.
- Puedes modificar el archivo de parámetros con `slam_params_file:=...`.

Ejemplo con archivo de parámetros explícito:

```bash
ros2 launch puzzlebot_navigation slam.launch.xml \
  headless:=false \
  slam_params_file:=/home/karinam/puzzlebot_ws/src/puzzlebot_ros2/puzzlebot_navigation/config/slam_toolbox.yaml
```

### 4. Guardar el mapa generado

Después de mapear el entorno con SLAM, puedes guardar el mapa para usarlo con Nav2:

```bash
ros2 run nav2_map_server map_saver_cli -f ~/puzzlebot_ws/src/puzzlebot_ros2/puzzlebot_navigation/maps/map
```

Esto actualiza los archivos `map.pgm` y `map.yaml` dentro de `puzzlebot_navigation/maps/`.

### 5. Ejecutar navegación con Nav2

Este flujo levanta:

- Gazebo con el robot
- `nav2_bringup`
- AMCL
- costmaps, planner, controller y behavior tree de Nav2
- RViz para enviar metas de navegación

Comando:

```bash
ros2 launch puzzlebot_navigation nav2.launch.xml headless:=false
```

Si quieres usar otro mapa:

```bash
ros2 launch puzzlebot_navigation nav2.launch.xml \
  headless:=false \
  map_path:=/home/karinam/puzzlebot_ws/src/puzzlebot_ros2/puzzlebot_navigation/maps/map.yaml
```

Una vez abierto RViz:

1. Usa `2D Pose Estimate` para indicar la pose inicial del robot.
2. Usa `Nav2 Goal` para enviar un objetivo de navegación.

## Launch files principales

| Archivo | Propósito |
| --- | --- |
| `puzzlebot_description/launch/puzzlebot_description.launch.xml` | Publica la descripción del robot y opcionalmente abre RViz, GUI de joints o una simulación vacía. |
| `puzzlebot_gazebo/launch/puzzlebot_gazebo.launch.xml` | Abre Gazebo con el mundo del laberinto, spawnea el robot y activa el bridge ROS-Gazebo. |
| `puzzlebot_navigation/launch/slam_core.launch.xml` | Ejecuta el núcleo de SLAM: `slam_toolbox`, RViz y teleoperación. |
| `puzzlebot_navigation/launch/slam.launch.xml` | Combina simulación + SLAM. |
| `puzzlebot_navigation/launch/nav2_core.launch.xml` | Ejecuta el núcleo de navegación con Nav2 y RViz usando un mapa. |
| `puzzlebot_navigation/launch/nav2.launch.xml` | Combina simulación + Nav2. |

## Flujo recomendado de uso

Para probar todo el stack de forma ordenada:

1. Compila el workspace y haz `source` del entorno.
2. Lanza `slam.launch.xml` para mapear el entorno.
3. Guarda el mapa en `puzzlebot_navigation/maps/`.
4. Lanza `nav2.launch.xml` usando el mapa guardado.
5. En RViz, fija la pose inicial y envía metas de navegación.

## Observaciones

- `slam.launch.xml` y `nav2.launch.xml` ya incluyen la simulación, así que no hace falta correr `puzzlebot_gazebo` por separado para esos flujos.
- `nav2.launch.xml` tiene `headless:=true` por defecto en el archivo, por lo que normalmente conviene correrlo con `headless:=false` si quieres ver Gazebo.
- `slam_core.launch.xml` y `nav2_core.launch.xml` están pensados para reutilizar la parte central del stack incluso fuera del flujo completo de simulación.
