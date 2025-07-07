Proyecto: Despliegue Node.js + PostgreSQL con Docker y Ansible

Este proyecto muestra cómo desplegar una aplicación Node.js con base de datos PostgreSQL utilizando dos enfoques:
	1.	Docker Compose
	2.	Ansible para automatización del entorno

⸻

Estructura del Proyecto

devops/
├── desafio_ansible_dia6/
│   └── roles/
│       └── nginx/...
└── node-postgres-ansible/
    ├── files/
    │   └── node-postgres-compose/
    │       ├── docker-compose.yml
    │       └── backend/
    │           └── app.js
    ├── inventories/hosts.ini
    ├── playbook.yml
    └── roles/
        ├── node/
        └── postgres/


⸻

Parte 1: Docker Compose

Pasos ejecutados:
	1.	Creación de docker-compose.yml:

version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    environment:
      - DATABASE_URL=postgres://postgres:password@postgres:5432/mydb

  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:

	2.	Backend en Express:

const { Pool } = require('pg');
...

	3.	Errores enfrentados:

	•	❌ Port 5432 already allocated
	•	Solución: ejecutar sudo lsof -i :5432 y liberar el puerto
	•	❌ Conexión rechazada (ECONNREFUSED)
	•	Solución: corregir variables de entorno en app.js

⸻

Parte 2: Despliegue con Ansible

Objetivo

Automatizar el despliegue completo de la app (backend + postgres) en múltiples servidores.

playbook.yml

- name: Desplegar stack Node + PostgreSQL
  hosts: dev
  become: yes
  tasks:
    - name: Instalar Docker y Docker Compose
      apt:
        name: ["docker.io", "docker-compose"]
        state: present
        update_cache: yes

    - name: Crear carpeta del proyecto
      file:
        path: /home/ubuntu/node-postgres-compose
        state: directory
        owner: ubuntu
        group: ubuntu
        mode: '0755'

    - name: Copiar archivos del proyecto
      copy:
        src: ./files/node-postgres-compose/
        dest: /home/ubuntu/node-postgres-compose/
        owner: ubuntu
        group: ubuntu

    - name: Levantar contenedores con docker-compose
      shell: docker-compose up -d
      args:
        chdir: /home/ubuntu/node-postgres-compose

Errores comunes y soluciones
	•	❌ Could not find or access './node-postgres-compose/'
	•	✅ Solución: asegurarse que la ruta esté dentro de files/
	•	❌ No space left on device
	•	✅ Solución: liberar espacio en disco
	•	❌ port 5432 already allocated
	•	✅ Solución: detener procesos existentes con lsof + docker stop

⸻

¿Por qué usar Ansible?
	•	Facilita el despliegue en múltiples máquinas
	•	Repetible y versionable
	•	Se integra con herramientas CI/CD fácilmente

¿Por qué usar Docker Compose?
	•	Para entornos de desarrollo rápidos y aislados
	•	Permite levantar múltiples servicios (app + db) con un solo comando

⸻

Créditos

Creado por Andy, usando Ubuntu + Multipass + Bash + Docker + Ansible sobre un entorno DevOps educativo.

⸻

Próximos pasos
	•	Agregar pruebas automatizadas
	•	Añadir integración continua (GitHub Actions)
	•	Versionar imágenes con etiquetas específicas
