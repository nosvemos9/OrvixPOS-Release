# React + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Oxc](https://oxc.rs)
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/)

## React Compiler

The React Compiler is not enabled on this template because of its impact on dev & build performances. To add it, see [this documentation](https://react.dev/learn/react-compiler/installation).

## Expanding the ESLint configuration

If you are developing a production application, we recommend using TypeScript with type-aware lint rules enabled. Check out the [TS template](https://github.com/vitejs/vite/tree/main/packages/create-vite/template-react-ts) for information on how to integrate TypeScript and [`typescript-eslint`](https://typescript-eslint.io) in your project.


-----------------------------------------------------------------------


# OrvixPos — Satış Noktası Yönetim Sistemi (POS)

Bu doküman, OrvixPos yazılımının genel mimarisini, operasyonel yeteneklerini ve teknik altyapısını detaylandırmak amacıyla hazırlanmıştır. Projenin amacı; restoran ve kafeler için donanımla tam entegre, bulut tabanlı ve uzaktan yönetilebilir bir masaüstü POS çözümü sunmaktır.

1. Proje Özeti ve İş Modeli (Yönetim Özeti)
OrvixPos, bir restoran veya kafenin tüm uçtan uca operasyonlarını tek bir ekrandan yönetebilmesi için geliştirilmiş; donanımla tam konuşabilen, bulut tabanlı ve yapay zeka destekli bir masaüstü POS (Point of Sale) çözümüdür.

Standart, hantal ve sadece adisyon kesmeye yarayan yazılımların aksine OrvixPos;

Telefon çaldığında müşteriyi tanıyan Caller ID entegrasyonuyla sipariş hızını artırır,

Her siparişte (Örn: BigKing) reçetedeki köfteyi gramajına kadar hesaplayıp stoktan anlık düşer,

Gün sonlarında sıradan Z-Raporları yerine, satış verilerini analiz eden Yapay Zeka (AI) Raporları ile işletme sahibine "Hangi ürün ne zaman kampanya yapılmalı?" gibi stratejik akıl verir.

The Burger Mill gibi müşterilerimiz için tasarlanan bu sistem, Orvix'in yazılımı bir kerelik satmak yerine, donanım gereksinimlerini otomatik ayarlayan, kendi kendini güncelleyen (OTA) aylık/yıllık lisanslanabilir kurumsal bir SaaS ürünü olarak konumlanmasını sağlar.

2. Temel Modüller ve Mimari Detaylar
Aşağıdaki bölümde, sisteme baştan sona entegre edilen kritik özellikler önce Operasyonel İşleyiş (Yönetim/Kullanıcı gözünden), ardından Teknik Altyapı (Geliştiriciler gözünden) detaylandırılmıştır.

2.1. Satış Yönetimi ve Caller ID Entegrasyonu
Operasyonel İşleyiş: Kasiyerler, masa servislerini ve online/paket siparişleri tek bir "Komuta Merkezi"nden yönetir. Dükkanın telefonu çaldığı anda Caller ID modülü devreye girer; arayan müşterinin adı, adresi ve geçmiş siparişleri ekrana otomatik düşer. Böylece telefonda adres sorma karmaşası biter. Hızlı sipariş butonları (Hazırla, Ödendi) yoğun saatlerde spam tıklamalara karşı kendini kilitler.

Teknik Altyapı: React Router (HashRouter) üzerinde inşa edilen UI, Firebase Realtime Database/Firestore ile anlık senkronize çalışır. Caller ID cihazından gelen sinyaller özel bir donanım/API dinleyicisi ile yakalanıp React state'ine aktarılır. "Race Condition" hatalarını önlemek için sipariş durum değişikliklerinde Optimistic UI ve Debounce/Loading kilit mekanizmaları uygulanmıştır.

2.2. İş Zekası, AI Destekli Raporlar ve Z-Raporu
Operasyonel İşleyiş: Kasiyerin gün sonu çekmeceyi teslim etmesi için tasarlanan dar fişin ötesinde; işletme sahipleri, A4 ekran genişliğinde, kategori ve ödeme tipi bazlı detaylı Z-Raporları (Dashboard) alabilir. Raporlar & AI modülü, haftalık satış verilerini analiz ederek işletmeye özel tavsiyeler üretir (Örn: "Pazar günleri sıcak içecek satışları düşük, hamburger menülerinin yanına promosyon ekleyebilirsiniz").

Teknik Altyapı: Tarih aralığına göre dinamik filtrelenen veriler (orders), bir LLM (Büyük Dil Modeli / Örn: OpenAI) API'sine özel bir "Prompt Engineering" dizilimiyle gönderilir. Gelen analiz yanıtı UI'da render edilir. Z-Raporu dökümü, termal kağıt görünümünü bozmadan kurumsal bir Dashboard'a entegre edilmiştir.

2.3. Dinamik İşletme Ayarları ve Reçete (Stok) Motoru
Operasyonel İşleyiş: OrvixPos her dükkana göre esneyebilir. Ayarlar panelinden; işletmenin adı, çalışma saatleri, termal kağıt boyutu (58mm veya 80mm) değiştirilebilir. Menüdeki ürünler sadece isimden ibaret değildir; bir hamburger satıldığında ekmek adeti, köfte gramajı ve sos miktarı arka planda stoktan otomatik düşülür.

Teknik Altyapı: Dinamik yapılandırma verileri global state (Context/Redux) veya yerel veritabanında (Dexie.js) tutulur. Satış anında tetiklenen StokMotoru algoritması, ürün reçetelerini (recipes) hesaplayıp, await db.update(...) asenkron işlemleriyle envanter verilerini eksi (-) bakiyeye düşmeden hatasız günceller.

2.4. Donanım ve Termal Yazıcı İzolasyonu
Operasyonel İşleyiş: Kasiyer siparişi onayladığında Windows işletim sisteminin yavaşlatan "Yazdır" veya "PDF Kaydet" ekranları çıkmaz. Sistem, adisyonu arka planda işletmenin 80mm/58mm yazıcısından anında (sessizce) çıkartır.

Teknik Altyapı: Electron webContents.print({ silent: true }) API'si ile işletim sistemi diyalogları atlanmıştır. Yazdırılacak format, ReceiptTemplate.jsx adlı gizli bir bileşen içinde, @media print kuralları ve monospace fontlar ile donanıma uygun piksel hassasiyetinde hazırlanmıştır.

2.5. Kurumsal SaaS Güvenliği ve Otomatik Güncelleme (OTA)
Operasyonel İşleyiş: Müşterinin USB bellek veya teknik servis beklemesine gerek yoktur. Yeni bir özellik (Örn: Yemeksepeti entegrasyonu) eklediğimizde, uygulama açılışında arka planda güncellemeyi indirir ve yeniden başlatma onayı ister. Menü ve şifreler koruma altındadır; sistem Kiosk mantığıyla tam ekran (menüsüz) çalışır.

Teknik Altyapı: electron-updater ile GitHub Release bağlantısı kurularak OTA (Over-The-Air) altyapısı inşa edilmiştir. vite-plugin-javascript-obfuscator ile projedeki tüm React kodları şifrelenerek ticari sırlar korunmuş, Electron pencerelerinde varsayılan menüler (autoHideMenuBar) nükleer yöntemle kapatılıp güvenlik politikaları (CSP, Context Isolation) en üst seviyeye çıkarılmıştır.

---

## 3. Teknoloji Yığını (Tech Stack)
* **Frontend:** React.js, Vite, Tailwind CSS, Lucide Icons, React Router Dom (HashRouter)
* **Backend & Veritabanı:** Firebase (Authentication, Firestore), Dexie.js (Offline Fallback mekanizması)
* **Desktop Platform:** Electron.js
* **Paketleme ve Dağıtım:** Electron-Builder, NSIS (Windows Installer)

---

## 4. Geliştirici Ortamı ve Kurulum Talimatları

Projeyi yerel ortamda çalıştırmak ve test etmek için aşağıdaki adımları izleyin:

**1. Bağımlılıkların Kurulması:**
```bash
npm install
