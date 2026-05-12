# Diagram

## Container Diagram

![Container Diagram](images/Container_Diagram.png)

## Context Diagram

![Context Diagram](images/Context_Diagram.png)

## Deployment Diagram

![Deployment Diagram](images/Deployment_Diagram.png)

## MySawit Future Architecture

Jika sebelumnya, arsitektur MySawit masih memiliki berbagai kerentanan. Alhasil, kami melakukan pengembangan dengan melakukan shifting dari sekadar VPS biasa ke infrastruktur yang lebih cloud-native dan serverless. Mengenai hal itu, kami akan mengimplementasikan deployment dengan AWS ECS Fargate agar deployment tidak hanya terpaku di satu VPS, tetapi sudah “serverless” karena ekosistem penggunaan server sudah disesuaikan oleh AWS sendiri. Hal ini akhirnya dapat membuat sistem bersifat auto-scale dan zero-downtime deployment. 

Lalu, kami juga menambahkan di bagian depan, request dari frontend (React Vite) akan masuk dulu ke Cloudflare untuk mencegah adanya serangan DDoS. Hal ini akan dicegah dengan prinsip-prinsip yang ada diterapkan oleh cloudflare. Setelah itu, jika aman, request akan diteruskan ke Application Load Balancer (ALB) AWS. Penggunaan Application Load Balancer adalah untuk mendistribusikan beban trafik secara merata ke task atau container yang tersedia sehingga mencegah terjadinya overload pada satu titik. Setelah dari ALB, request masuk ke API Gateway yang berfungsi untuk mengurusi spesifik routing, rate limiting, dan verifikasi token JWT sebelum di-forward ke service yang tepat. Setiap service di sini juga sudah auto-scaled karena semua service di-deploy menggunakan AWS ECS Fargate yang sama.

Selain itu, hal krusial lainnya adalah komunikasi antar-service kami rombak total menjadi fully asynchronous dengan implementasi message broker, yakni RabbitMQ. Sebagai contoh, ketika service Panen atau Pengiriman melakukan approval, mereka nggak nge-hit API service Pembayaran secara langsung, tetapi cukup melakukan publish event, seperti harvest.approved, dan service Pembayaran cukup subscribe untuk triggering payroll. Hal ini kami lakukan untuk menghindari blocking dalam komunikasi antarservice sehingga sebuah service tidak perlu membuang waktu menunggu respon dari service lain. Selain itu, untuk kebutuhan pembacaan data lintas domain, misalnya service Kebun butuh data mandor, kami menerapkan replikasi data via RabbitMQ. Alhasil, sebuah service tidak perlu terus-menerus menembak API service utama hanya untuk membaca data, melainkan cukup mengambil dari database lokalnya sendiri yang datanya secara otomatis disinkronkan melalui message broker.

![Future Architecture Diagram](images/Future_Architecture_Diagram.png)
