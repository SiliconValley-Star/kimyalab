# 🚀 KimyaLab Hostinger Deployment Guide

## 📋 Genel Bakış

Bu rehber KimyaLab projenizi Hostinger'a deploy etmek için gerekli tüm adımları içerir.

### ✅ Hazırladığımız Dosyalar
- `backend/.env.production` - Backend production konfigürasyonu
- `.env.production` - Frontend production konfigürasyonu  
- `backend/scripts/production-deployment.ts` - Otomatik deployment script
- Enhanced package.json scripts - Production build komutları

---

## 🗄️ 1. DATABASE SETUP (PostgreSQL)

### 1.1 Hostinger PostgreSQL Database Oluşturma

1. **Hostinger Panel → Databases → Create PostgreSQL Database**
2. Database bilgileri:
   ```
   Database Name: kimyalab_db
   Username: kimyalab_user
   Password: [güçlü şifre oluşturun]
   ```
3. Connection string'i kopyalayın

### 1.2 Environment Variables Güncelleme

`backend/.env.production` dosyasındaki DATABASE_URL'i güncelleyin:
```bash
DATABASE_URL="postgresql://kimyalab_user:YOUR_PASSWORD@postgresql.hostinger.com:5432/kimyalab_db"
```

---

## ☁️ 2. CLOUDINARY SETUP

### 2.1 Cloudinary Hesabı Oluşturma

1. [Cloudinary.com](https://cloudinary.com)'da hesap açın (ücretsiz)
2. Dashboard'dan şu bilgileri alın:
   - Cloud name
   - API Key  
   - API Secret

### 2.2 Environment Variables Güncelleme

**Backend (.env.production):**
```bash
FORCE_MOCK_UPLOADS=false
CLOUDINARY_CLOUD_NAME=your-actual-cloud-name
CLOUDINARY_API_KEY=your-actual-api-key
CLOUDINARY_API_SECRET=your-actual-api-secret
```

**Frontend (.env.production):**
```bash
VITE_CLOUDINARY_CLOUD_NAME=your-actual-cloud-name
```

---

## 🔧 3. BACKEND DEPLOYMENT

### 3.1 Hostinger Node.js App Oluşturma

1. **Hostinger Panel → Node.js → Create**
2. Settings:
   ```
   Node.js Version: 18.x veya üzeri
   Startup File: dist/index.js
   Application Root: /backend
   ```

### 3.2 Git Repository Bağlama

1. GitHub'da repository oluşturun
2. Kodu push edin:
   ```bash
   git add .
   git commit -m "Production ready deployment"
   git push origin main
   ```

### 3.3 Environment Variables Ekleme

Hostinger Node.js panel'inde şu environment variables'ları ekleyin:

```bash
NODE_ENV=production
PORT=8080
DATABASE_URL=postgresql://kimyalab_user:PASSWORD@postgresql.hostinger.com:5432/kimyalab_db
JWT_SECRET=your-super-strong-jwt-secret
ADMIN_EMAIL=admin@kimyalab.com
ADMIN_PASSWORD=your-strong-admin-password
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
FORCE_MOCK_UPLOADS=false
FRONTEND_URL=https://kimyalab.com
CORS_ORIGINS=https://kimyalab.com,https://www.kimyalab.com
```

### 3.4 Deploy Scripts Çalıştırma

Terminal'de backend klasöründe:

```bash
# Production build oluştur
npm run production:build

# Database migration ve setup
npm run production:deploy

# Production server başlat
npm run production:start
```

---

## 🎨 4. FRONTEND DEPLOYMENT

### 4.1 Production Build Oluşturma

```bash
# Environment variables'ları güncelle
cp .env.production .env

# Dependencies install
npm install

# Production build
npm run build
```

### 4.2 Hostinger Static Site Upload

1. `dist/` klasörünü Hostinger File Manager'a upload edin
2. Public_html klasörüne kopyalayın
3. Index.html'in root'ta olduğundan emin olun

### 4.3 API URL Güncellemeleri

`.env.production`'da API URL'leri gerçek backend URL'leri ile güncelleyin:
```bash
VITE_API_URL=https://api-kimyalab.hostinger-site.com
VITE_WS_URL=wss://api-kimyalab.hostinger-site.com
```

---

## 🌐 5. DOMAIN & SSL CONFIGURATION

### 5.1 Domain Bağlama

1. **Hostinger Panel → Domains → Manage**
2. DNS ayarlarını yapın:
   ```
   A Record: @ → Server IP
   CNAME: www → domain.com
   CNAME: api → backend-app-url
   ```

### 5.2 SSL Certificate

1. **Hostinger Panel → SSL**
2. Let's Encrypt SSL'i aktifleştirin
3. HTTPS redirect'i etkinleştirin

---

## 🔧 6. PRODUCTION OPTIMIZATION

### 6.1 Backend Optimizations

`backend/src/server.ts`'de production ayarları:
```typescript
if (process.env.NODE_ENV === 'production') {
  app.use(compression())
  app.use(helmet())
  app.set('trust proxy', 1)
}
```

### 6.2 Database Optimizations

```bash
# Connection pooling
DATABASE_URL="postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=20"
```

### 6.3 Frontend Optimizations

Vite build optimizasyonları:
```javascript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@mui/material', '@emotion/react']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  }
})
```

---

## 🔍 7. TESTING & MONITORING

### 7.1 Deployment Testing Checklist

- [ ] **Database Connection**: Test PostgreSQL bağlantısı
- [ ] **API Endpoints**: Tüm endpoint'leri test edin
- [ ] **File Upload**: Cloudinary upload testi
- [ ] **Authentication**: Admin login testi
- [ ] **Search Functionality**: Arama sistemi testi
- [ ] **Image Display**: Fotoğraf görüntüleme testi
- [ ] **Mobile Responsive**: Mobil uyumluluk
- [ ] **SSL Certificate**: HTTPS kontrolü

### 7.2 Monitoring Setup

```bash
# Health check endpoint
curl -f https://api.kimyalab.com/health

# Database monitoring
npm run logs
```

### 7.3 Performance Testing

- **Page Speed**: Google PageSpeed Insights
- **API Response**: Postman/Insomnia ile endpoint testleri
- **Database Performance**: Prisma metrics
- **Upload Speed**: Cloudinary dashboard

---

## 🚨 8. TROUBLESHOOTING

### 8.1 Common Issues

**❌ Database Connection Error:**
```bash
# Check connection string
echo $DATABASE_URL

# Test connection
npm run db:generate
npx prisma migrate deploy
```

**❌ Cloudinary Upload Error:**
```bash
# Check credentials
echo $CLOUDINARY_CLOUD_NAME
echo $CLOUDINARY_API_KEY

# Test upload
npm test -- cloudinary
```

**❌ Frontend API Connection:**
```bash
# Check CORS settings
# Check VITE_API_URL
# Check network tab in browser
```

### 8.2 Debug Commands

```bash
# Backend logs
npm run logs

# Database status
npx prisma migrate status

# Health check
npm run healthcheck

# Build verification
npm run production:build
```

---

## 📊 9. POST-DEPLOYMENT CHECKLIST

### 9.1 Security Checklist

- [ ] Admin şifresini değiştirin
- [ ] JWT secret'i güncelleli
- [ ] Database kullanıcı şifresi güçlü
- [ ] CORS origins doğru
- [ ] Rate limiting aktif
- [ ] Helmet security headers
- [ ] SSL certificate aktif

### 9.2 Functionality Test

- [ ] Ana sayfa loading
- [ ] Ürün listesi görüntüleme
- [ ] Ürün detay sayfası
- [ ] Arama functionality
- [ ] Admin panel erişimi
- [ ] Ürün ekleme/düzenleme
- [ ] Fotoğraf upload
- [ ] Kategori sistemi
- [ ] Contact form

### 9.3 Performance Check

- [ ] Page loading < 3 seconds
- [ ] API response < 500ms
- [ ] Image loading optimized
- [ ] Mobile responsive
- [ ] SEO meta tags
- [ ] Google Analytics (opsiyonel)

---

## 🔄 10. BACKUP & MAINTENANCE

### 10.1 Database Backup

```bash
# Manual backup
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql

# Automated backup (cron job)
0 2 * * * pg_dump $DATABASE_URL > /backups/daily_$(date +%Y%m%d).sql
```

### 10.2 Code Updates

```bash
# Development'da değişiklik yap
git add .
git commit -m "Update feature"
git push origin main

# Production'da güncelle
git pull origin main
npm run production:build
npm restart
```

### 10.3 Monitoring

- **Uptime**: UptimeRobot veya benzeri
- **Performance**: Google Analytics
- **Errors**: Sentry.io integration
- **Database**: PostgreSQL logs

---

## 📞 11. SUPPORT & RESOURCES

### 11.1 Documentation Links

- [Hostinger Node.js Hosting](https://support.hostinger.com/en/articles/1583579-how-to-use-node-js)
- [Prisma Production Guide](https://www.prisma.io/docs/guides/deployment)
- [Cloudinary Documentation](https://cloudinary.com/documentation)
- [Vite Build Guide](https://vitejs.dev/guide/build.html)

### 11.2 Emergency Contacts

- **Database Issues**: Hostinger Support
- **Domain/DNS**: DNS Management Panel
- **SSL Issues**: Let's Encrypt Support
- **Cloudinary**: Cloudinary Support Portal

---

## 🎉 DEPLOYMENT TAMAMLANDI!

Başarılı deployment sonrası:

1. **Domain**: https://kimyalab.com
2. **API**: https://api.kimyalab.com  
3. **Admin Panel**: https://kimyalab.com/admin
4. **Default Admin**: admin@kimyalab.com / [admin şifreniz]

### 🔐 GÜVENLİK HATIRLATMASI
- Admin şifresini mutlaka değiştirin
- Database backup'ını alın
- SSL certificate'i kontrol edin
- Security headers'ları doğrulayın

---

## 🤝 DESTEK

Bu deployment guide ile ilgili sorularınız için:
- GitHub Issues
- Email: technical-support@kimyalab.com
- Documentation: /docs

**Başarılı deployment'lar! 🚀**