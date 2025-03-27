# Practica-Modulo-VII

==============================
En el Servidor (Red Hat - 10.0.0.34):

# 1. Instalar NFS
sudo dnf install nfs-utils -y

# 2. Crear directorio y archivos
sudo mkdir /OS3
cd /OS3
sudo touch Adrian{1..100}.txt

# 3. Asignar permisos
sudo chmod -R 777 /OS3
sudo chown nobody:nobody /OS3

# 4. Configurar exports
echo "/OS3 10.0.0.63(rw,sync,no_root_squash)" | sudo tee -a /etc/exports

# 5. Habilitar servicios y aplicar cambios
sudo systemctl enable --now nfs-server rpcbind
sudo exportfs -a

# 6. Configurar firewall (opcional, si hay bloqueos)
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload


En el Cliente (Linux Mint - 10.0.0.63):

# 1. Instalar cliente NFS
sudo apt update && sudo apt install nfs-common -y

# 2. Crear directorio de montaje
sudo mkdir -p /mnt/OS3

# 3. Montar manualmente (prueba inicial)
sudo mount -t nfs 10.0.0.34:/OS3 /mnt/OS3

# 4. Verificar archivos
ls /mnt/OS3  # Deberías ver Adrian1.txt ... Adrian100.txt

# 5. Configurar montaje automático en fstab
echo "10.0.0.34:/OS3 /mnt/OS3 nfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab

# 6. Probar montaje automático
sudo mount -a

# 7. Reiniciar y verificar (después del reinicio)
sudo reboot


Después de Reiniciar el Cliente:
bash
Copy
# Verificar montaje automático
df -h | grep OS3  # Debe mostrar /mnt/OS3 montado

# Listar archivos compartidos
ls /mnt/OS3

---------------------------------------------------------------------
Práctica 2: Creación de FileServer compatible con Windows utilizando SAMBA
---------------------------------------------------------------------

[VM Server – Tu distro Linux]
-----------------------------------------------------
1. Instalar y habilitar Samba:
   sudo dnf install -y samba samba-common samba-client
   sudo systemctl enable --now smb nmb

2. Crear la carpeta compartida y 100 archivos:
   sudo mkdir -p /srv/samba/compartida
   for i in {1..100}; do
       echo "Contenido Samba $i" > /srv/samba/compartida/adrian${i}.txt
   done

3. Crear grupo y usuario para Samba:
   sudo groupadd smbusers
   sudo useradd -M -s /sbin/nologin smbuser
   sudo passwd smbuser         # Define la contraseña deseada
   sudo smbpasswd -a smbuser   # Agrega el usuario a Samba

4. Asignar permisos a la carpeta compartida:
   sudo chown -R smbuser:smbusers /srv/samba/compartida
   sudo chmod -R 770 /srv/samba/compartida

5. Configurar Samba (editar el archivo /etc/samba/smb.conf):
   sudo nano /etc/samba/smb.conf

   Agrega al final del archivo:
   [compartida]
      path = /srv/samba/compartida
      valid users = smbuser
      browsable = yes
      writable = yes
      guest ok = no
      create mask = 0660
      directory mask = 0770

   Guarda los cambios y reinicia Samba:
   sudo systemctl restart smb nmb

[VM Windows 10]
-----------------------------------------------------
1. Mapear la carpeta compartida:
   - Abre el Explorador de archivos.
   - Haz clic derecho en "Este Equipo" y selecciona "Conectar unidad de red".
   - Selecciona una letra (por ejemplo, Z:) y en la ruta ingresa:
       \\<IP_DEL_SERVIDOR>\compartida
   - Marca la opción "Conectar con credenciales diferentes".
   - Ingresa el usuario "smbuser" y la contraseña definida en el paso anterior.

2. Verificar archivos:
   Accede a la unidad mapeada y verifica que aparezcan los archivos adrian1.txt a adrian100.txt.

3. Editar un archivo:
   Abre el archivo adrian99.txt, agrega la línea:
       el zumzum de la carabela
   Guarda los cambios.

[VM Server – Verificación en Linux]
-----------------------------------------------------
1. Verificar la modificación del archivo:
   cat /srv/samba/compartida/adrian99.txt
   (Deberías ver la frase "el zumzum de la carabela")

---------------------------------------------------------------------
Práctica 3: Creación de Controlador de Dominio con cliente Windows usando Samba4
---------------------------------------------------------------------

[VM Server – Tu distro Linux]
-----------------------------------------------------
1. Instalar Samba DC y detener servicios que puedan interferir:
   sudo dnf install -y samba samba-dc samba-client
   sudo systemctl stop smb nmb winbind

2. Provisionar el dominio:
   sudo samba-tool domain provision --use-rfc2307 --interactive

   Durante la provisión, ingresa lo siguiente:
   - Realm: SO3.inet
   - Domain: SO3
   - Server Role: dc
   - DNS Backend: SAMBA_INTERNAL
   - Administrator Password: (elige una contraseña segura)

3. Crear el usuario de dominio "lanegracubana":
   (Utiliza la matrícula como contraseña, por ejemplo, 12345678)
   sudo samba-tool user create lanegracubana --random-password
   sudo samba-tool user setpassword lanegracubana --newpassword="12345678"
   sudo smbpasswd -a lanegracubana

[VM Windows 10]
-----------------------------------------------------
1. Unir Windows al dominio:
   - Haz clic derecho en "Este Equipo" y selecciona "Propiedades".
   - Ve a "Configuración avanzada del sistema" y luego haz clic en "Cambiar configuración" en la sección de nombre de equipo.
   - Haz clic en "Cambiar" y selecciona "Dominio". Ingresa:
         SO3.inet
   - Cuando se soliciten credenciales, introduce:
         Usuario: SO3\lanegracubana
         Contraseña: 12345678
   - Acepta y reinicia Windows.
   - Al iniciar sesión, selecciona el dominio SO3 y el usuario lanegracubana.

---------------------------------------------------------------------
Resumen de Máquinas y Acciones
---------------------------------------------------------------------

[VM Server – Tu distro Linux]
- Práctica 1 (NFS):
  • Instalar NFS: sudo dnf install -y nfs-utils
  • Crear directorio /srv/nfs/OS3 y 100 archivos con un loop for
  • Asignar permisos: chown a nobody, chmod 777
  • Configurar /etc/exports y reiniciar nfs-server

- Práctica 2 (Samba):
  • Instalar Samba: sudo dnf install -y samba samba-common samba-client
  • Crear carpeta /srv/samba/compartida y 100 archivos
  • Crear grupo y usuario (smbuser) y asignar permisos
  • Configurar /etc/samba/smb.conf y reiniciar servicios smb y nmb

- Práctica 3 (Samba4 - Dominio):
  • Instalar Samba DC: sudo dnf install -y samba samba-dc samba-client
  • Detener servicios (smb, nmb, winbind)
  • Provisionar dominio con: sudo samba-tool domain provision --use-rfc2307 --interactive
  • Crear usuario de dominio: lanegracubana

[VM Linux Cliente – Tu distro Linux de preferencia]
- Práctica 1 (NFS):
  • Instalar cliente NFS: sudo dnf install -y nfs-utils
  • Crear punto de montaje /mnt/nfs_OS3, montar el recurso, y configurar /etc/fstab para montaje persistente

[VM Windows 10]
- Práctica 2 (Samba):
  • Mapear la unidad de red con la ruta \\<IP_DEL_SERVIDOR>\compartida
  • Editar el archivo adrian99.txt para agregar "el zumzum de la carabela"

- Práctica 3 (Dominio):
  • Unir al dominio SO3.inet usando el usuario SO3\lanegracubana y reiniciar Windows

==============================
Fin de las instrucciones
==============================
