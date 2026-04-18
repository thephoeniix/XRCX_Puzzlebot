# XRCX_Puzzlebot

> Stack de navegación autónoma en ROS 2 para el robot Puzzlebot, con soporte para SLAM, simulación en Gazebo y navegación con Nav2.


## Integrantes

| Nombre | GitHub |
|--------|--------|
| Sofía Blanco Prigmore | [AifosWhite](https://github.com/AifosWhite) |
| Josué Aldemar Garduño Gómez  | [aldemar3002](https://github.com/aldemar3002) |
| Karina Fernanda Maldonado Murillo  | [thephoeniix](https://github.com/thephoeniix) |
| Roberto Carlos Pedraza MIranda | [RoberttCap](https://github.com/RoberttCap) |



## Descripción general

Este repositorio contiene el stack de navegación en ROS 2 para el robot **Puzzlebot**, estructurado en tres paquetes con responsabilidades claramente separadas:

- `puzzlebot_description` — define cómo es el robot (URDF, meshes, frames).
- `puzzlebot_gazebo` — define cómo se ejecuta el robot en simulación (Gazebo, bridge, spawn).
- `puzzlebot_navigation2` — define cómo se comporta el stack de SLAM y navegación (Nav2, AMCL, mapas).

