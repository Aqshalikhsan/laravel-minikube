# Laravel Deployment on Kubernetes (Minikube)

Dokumentasi ini berisi panduan teknis lengkap untuk melakukan deployment aplikasi Laravel ke cluster Kubernetes menggunakan Minikube dan Docker Desktop pada sistem operasi Windows.

---

## Prasyarat

Pastikan perangkat lunak berikut telah terinstal:

- PHP 8.3 atau versi terbaru
- Composer
- Docker Desktop
- Minikube
- Kubectl
- Laragon (opsional, untuk pengujian lokal)

---

## 1. Setup Proyek Laravel Lokal

```bash
cd C:\Users\aksal\Documents
mkdir kubernetes-laravel
cd kubernetes-laravel

composer create-project laravel/laravel laravel-minikube
cd laravel-minikube

cp .env.example .env
php artisan key:generate
````

---

## 2. Konfigurasi Environment

Edit file `.env`:

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel_k8s
DB_USERNAME=root
DB_PASSWORD=
```

Catatan:

* `DB_HOST=mysql` digunakan agar Laravel dapat terhubung ke service MySQL di dalam Kubernetes.

---

## 3. Pengujian Lokal (Laragon)

Pastikan aplikasi berjalan sebelum masuk ke tahap containerization.

Langkah:

1. Jalankan Laragon → klik Start All
2. Jalankan migrasi:

```bash
php artisan migrate
```

3. Jalankan aplikasi:

```bash
php artisan serve
```

4. Akses di browser:

```
http://127.0.0.1:8000
```

---

## 4. Konfigurasi Dockerfile

Buat file `Dockerfile` di root project:

```dockerfile
FROM php:8.3-apache

RUN docker-php-ext-install pdo pdo_mysql
RUN a2enmod rewrite

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/000-default.conf
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf

COPY --chown=www-data:www-data . /var/www/html

RUN chmod -R 775 /var/www/html/storage /var/www/html/bootstrap/cache

EXPOSE 80
```

---

## 5. Inisialisasi Kubernetes (Minikube)

Pastikan Docker Desktop dalam keadaan aktif.

```bash
minikube start --driver=docker
kubectl get nodes
```

Jika berhasil:

```
minikube   Ready
```

---

## 6. Setup Kubernetes Manifest

Buat folder:

```bash
mkdir k8s
cd k8s
```

Buat file:

* laravel.yaml
* mysql.yaml
* service.yaml

Deploy:

```bash
kubectl apply -f .
kubectl get pods
kubectl get svc
```

---

## 7. Build dan Load Image ke Minikube

Kembali ke root project:

```bash
cd ..
docker build -t laravel-app .
minikube image load laravel-app
```

Verifikasi:

```bash
minikube ssh
docker images
```

Pastikan terdapat:

```
laravel-app   latest
```

---

## 8. Menjalankan Aplikasi

```bash
minikube service laravel-nodeport
```

---

## 9. Migrasi Database di Pod

```bash
kubectl get pods
kubectl exec -it <nama-pod-laravel> -- bash

php artisan migrate
```

---

## 10. Update Image

Jika ada perubahan kode:

```bash
docker build -t laravel-app .
minikube image load laravel-app
minikube service laravel-nodeport
```

---

## 11. Manajemen Skalabilitas

Scale up:

```bash
kubectl scale deployment laravel --replicas=5
```

Scale down:

```bash
kubectl scale deployment laravel --replicas=2
```

Monitoring:

```bash
kubectl get pods -w
```

---

## 12. Arsitektur Sistem

```
User → Service (NodePort) → Laravel Pods (Replicas) → MySQL Pod
```

Kubernetes menangani:

* Load balancing
* Service discovery
* Horizontal scaling

---

## 13. Troubleshooting

Hapus deployment:

```bash
kubectl delete deployment laravel
```

Hapus image:

```bash
docker rmi laravel-app
```

Cek image di Minikube:

```bash
minikube ssh
docker images
```

Restart Minikube:

```bash
minikube delete --all --purge
docker rm -f minikube
```

---

## 14. Reset Total (Clean Environment)

```bash
kubectl delete all --all

minikube stop
minikube delete --all --purge

docker system prune -a --volumes
docker builder prune -a

rmdir /s /q C:\Users\aksal\.minikube\cache
```

---

## 15. Membersihkan Temporary File Windows

```
Win + R → %temp% → hapus semua file
```

---

## Hasil

Dengan langkah ini:

* Laravel berjalan dalam container Docker
* Deployment di Kubernetes (Minikube)
* Load balancing melalui Service
* Skalabilitas menggunakan replicas
* Simulasi environment production secara lokal
