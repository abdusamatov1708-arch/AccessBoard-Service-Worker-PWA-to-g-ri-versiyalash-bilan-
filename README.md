# AccessBoard-Service-Worker-PWA-to-g-ri-versiyalash-bilan-
Loyiha fayllari tuzilmasi:
Plaintext
accessboard-pwa/
├── index.html
├── style.css
├── script.js
├── manifest.json
└── sw.js
1. manifest.json
PWA uchun ilova nomi, boshlang'ich sahifa, ko'rinish rejimi va piktogrammalar (icons) sozlangan fayl:

JSON
{
  "name": "AccessBoard PWA",
  "short_name": "AccessBoard",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#f3f4f6",
  "theme_color": "#111827",
  "icons": [
    {
      "src": "icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
2. sw.js (Service Worker)
Bu faylda versiyalangan kesh nomi, install paytida skipWaiting() va activate paytida eski keshni tozalash mantiqi yozilgan:

JavaScript
const CACHE_VERSION = 'accessboard-v1'; // Har bir deploy'da o'zgarib turadigan versiyalangan nom
const ASSETS_TO_CACHE = [
  './index.html',
  './style.css',
  './script.js',
  './manifest.json'
];

// 1. Install hodisasi: fayllarni keshga yig'ish va darhol faollashtirish
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_VERSION).then((cache) => {
      return cache.addAll(ASSETS_TO_CACHE);
    })
  );
  // Kutish bosqichini o'tkazib yuborib, darhol faollashtirish
  self.skipWaiting();
});

// 2. Activate hodisasi: eski versiyadagi keshni tozalab tashlash
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (cacheName !== CACHE_VERSION) {
            console.log('Eski kesh tozalanmoqda:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
  // Yangi Service Worker darhol sahifalarni boshqarishini ta'minlash
  self.clients.claim();
});

// 3. Fetch hodisasi: tarmoq yoki keshdan ma'lumot olish
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cachedResponse) => {
      if (cachedResponse) {
        return cachedResponse;
      }
      return fetch(event.request);
    })
  );
});
3. script.js (Service Worker'ni ro'yxatdan o'tkazish)
Brauzerda Service Worker ishga tushishini ta'minlovchi asosiy JavaScript kodi:

JavaScript
// Service Worker'ni ro'yxatdan o'tkazish
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('./sw.js')
      .then((registration) => {
        console.log('Service Worker muvaffaqiyatli ro‘yxatdan o‘tdi:', registration.scope);
      })
      .catch((error) => {
        console.error('Service Worker xatosi:', error);
      });
  });
}
4. index.html
PWA manifestini ulagich va asosiy tuzilmaga ega sahifa:

HTML
<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AccessBoard PWA</title>
  <!-- Manifestni ulash -->
  <link rel="manifest" href="manifest.json">
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <main class="container">
    <h1>AccessBoard PWA Taxtasi</h1>
    <p>Bu ilova offline (internet bo'lmaganda ham) ishlaydi va PWA sifatida o'rnatilishi mumkin.</p>
  </main>

  <script src="script.js"></script>
</body>
</html>
5. style.css
Oddiy va qulay dizayn:

CSS
body {
  font-family: system-ui, -apple-system, sans-serif;
  background-color: #f3f4f6;
  color: #111827;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

.container {
  background: #ffffff;
  padding: 2rem;
  border-radius: 8px;
  border: 2px solid #d1d5db;
  text-align: center;
  max-width: 400px;
}
