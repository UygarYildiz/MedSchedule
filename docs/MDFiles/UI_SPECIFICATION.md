# MedSchedule UI/UX Spesifikasyonu

Bu doküman, MedSchedule projesinin kullanıcı arayüzü tasarımını ve kullanıcı deneyimi prensiplerini detaylandırır.

## Tasarım Sistemi

### Renk Paleti

#### Ana Renkler
- **Primary**: `#2196F3` (Mavi)
  - Hover: `#1976D2`
  - Active: `#1565C0`
- **Secondary**: `#FF4081` (Pembe)
  - Hover: `#F50057`
  - Active: `#C51162`
- **Success**: `#4CAF50` (Yeşil)
- **Warning**: `#FFC107` (Sarı)
- **Error**: `#F44336` (Kırmızı)
- **Info**: `#2196F3` (Mavi)

#### Nötr Renkler
- **Background**: `#FFFFFF`
- **Surface**: `#F5F5F5`
- **Border**: `#E0E0E0`
- **Text Primary**: `#212121`
- **Text Secondary**: `#757575`
- **Text Disabled**: `#9E9E9E`

### Tipografi

#### Font Ailesi
- **Ana Font**: Inter
- **Başlık Font**: Poppins
- **Monospace**: JetBrains Mono

#### Font Boyutları
- **h1**: 32px / 40px line-height
- **h2**: 24px / 32px line-height
- **h3**: 20px / 28px line-height
- **h4**: 16px / 24px line-height
- **Body 1**: 16px / 24px line-height
- **Body 2**: 14px / 20px line-height
- **Caption**: 12px / 16px line-height

### Boşluk ve Grid

#### Boşluk Birimleri
- **xs**: 4px
- **sm**: 8px
- **md**: 16px
- **lg**: 24px
- **xl**: 32px
- **2xl**: 48px

#### Grid Sistemi
- 12 kolonlu grid
- Gutter: 24px
- Margin: 24px
- Breakpoints:
  - xs: 0px
  - sm: 600px
  - md: 960px
  - lg: 1280px
  - xl: 1920px

### Bileşenler

#### Butonlar

##### Primary Button
```css
{
    height: 40px;
    padding: 0 24px;
    border-radius: 20px;
    font-weight: 500;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}
```

##### Secondary Button
```css
{
    height: 36px;
    padding: 0 16px;
    border-radius: 18px;
    font-weight: 500;
    border: 2px solid;
}
```

##### Icon Button
```css
{
    width: 40px;
    height: 40px;
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

#### Form Elemanları

##### Text Input
```css
{
    height: 48px;
    padding: 0 16px;
    border-radius: 8px;
    border: 1px solid var(--border-color);
    background: var(--background);
}
```

##### Select
```css
{
    height: 48px;
    padding: 0 16px;
    border-radius: 8px;
    border: 1px solid var(--border-color);
    background: var(--background);
}
```

##### Checkbox
```css
{
    width: 20px;
    height: 20px;
    border-radius: 4px;
    border: 2px solid var(--border-color);
}
```

## Sayfa Tasarımları

### Giriş Sayfası (Login)

#### Layout
- Merkezi konumlandırılmış login formu
- Logo üstte
- Form alanları:
  - Kullanıcı adı
  - Parola
  - "Beni Hatırla" checkbox
  - Giriş yap butonu
- "Parolamı Unuttum" linki

#### Mobil Görünüm
- Tam ekran form
- Dikey düzen
- Touch-friendly input alanları

### Ana Dashboard

#### Layout
- Sol kenar çubuğu (sidebar) navigasyon
- Üst çubuk (header):
  - Kullanıcı profili
  - Bildirimler
  - Hızlı eylemler
- Ana içerik alanı:
  - Özet kartları
  - Grafik ve istatistikler
  - Son aktiviteler

#### Mobil Görünüm
- Gizlenebilir sidebar
- Alt navigasyon çubuğu
- Kartlar tam genişlik
- Kaydırılabilir grafikler

### Çizelge Görünümü

#### Layout
- Tam ekran takvim görünümü
- Üst toolbar:
  - Tarih navigasyonu
  - Görünüm seçenekleri
  - Filtreler
- Sol panel:
  - Personel listesi
  - Departman filtreleri
- Sağ panel (detay görünümü):
  - Vardiya detayları
  - Hızlı düzenleme

#### Mobil Görünüm
- Liste/Gün görünümü
- Gizlenebilir paneller
- Bottom sheet detay görünümü
- Touch-optimized kontroller

### Personel Yönetimi

#### Layout
- Tablo görünümü
- Üst toolbar:
  - Arama
  - Filtreler
  - "Yeni Ekle" butonu
- Detay görünümü:
  - Yan panel veya modal
  - Form düzeni
  - İlişkili veriler (vardiyalar, tercihler)

#### Mobil Görünüm
- Liste görünümü
- Tam ekran detay sayfası
- Pull-to-refresh
- Infinite scroll

## Etkileşim Tasarımı

### Animasyonlar

#### Geçiş Animasyonları
```css
{
    transition: all 0.3s ease;
}
```

#### Sayfa Geçişleri
- Fade in/out
- Slide için yön bazlı animasyonlar
- Modal için scale animasyonu

#### Yükleme Durumları
- Skeleton loading
- Progress spinners
- Pulse animasyonu

### Geri Bildirim

#### Toast Bildirimleri
- Sağ üst köşede
- 3 saniye görünür
- Kapatılabilir
- Tip bazlı renkler (success, error, warning, info)

#### Form Doğrulama
- Anlık doğrulama
- Hata mesajları input altında
- Başarı/hata durumu görsel göstergeleri

#### Eylem Onayları
- Modal dialog
- Tehlikeli eylemler için ek onay
- Geri alınabilir eylemler için undo özelliği

## Erişilebilirlik

### WCAG 2.1 Uyumluluğu

#### Klavye Navigasyonu
- Tüm etkileşimler için klavye desteği
- Görünür focus indikatörleri
- Mantıklı tab sırası

#### Ekran Okuyucu Desteği
- ARIA etiketleri
- Alt metinleri
- Anlamlı başlık hiyerarşisi

#### Renk Kontrastı
- Minimum 4.5:1 kontrast oranı
- Renk körü dostu tasarım
- Yüksek kontrast modu desteği

## Responsive Tasarım

### Breakpoint Stratejisi

#### Desktop First
```css
/* XL - 1920px ve üzeri */
@media (max-width: 1919px) {
    /* LG styles */
}

@media (max-width: 1279px) {
    /* MD styles */
}

@media (max-width: 959px) {
    /* SM styles */
}

@media (max-width: 599px) {
    /* XS styles */
}
```

#### Özel Breakpoint'ler
- Takvim görünümü: 1200px
- Detay paneli: 1400px
- Tablo görünümü: 800px

### Layout Adaptasyonu

#### Grid Sistemi
```css
.grid {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    gap: 24px;
}

@media (max-width: 959px) {
    .grid {
        grid-template-columns: repeat(8, 1fr);
    }
}

@media (max-width: 599px) {
    .grid {
        grid-template-columns: repeat(4, 1fr);
    }
}
```

#### Responsive Utilities
```css
.hide-on-mobile {
    @media (max-width: 599px) {
        display: none;
    }
}

.show-on-mobile {
    @media (min-width: 600px) {
        display: none;
    }
}
```

## Performans Optimizasyonu

### Görsel Optimizasyonu

#### Resim Boyutları
- Hero görselleri: max 1920px genişlik
- Thumbnail: 200px x 200px
- Avatar: 40px x 40px
- İkonlar: SVG veya 24px x 24px

#### Lazy Loading
```html
<img 
    loading="lazy"
    src="image.jpg"
    srcset="image-sm.jpg 300w, image-md.jpg 600w, image-lg.jpg 1200w"
    sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
    alt="Description"
>
```

### İnteraktif Performans

#### Debounce/Throttle
```javascript
// Arama input'u için debounce
const debounceSearch = debounce((value) => {
    searchAPI(value);
}, 300);

// Takvim scroll'u için throttle
const throttleScroll = throttle((event) => {
    updateCalendarView(event);
}, 100);
```

#### Virtualization
- Uzun listeler için virtual scroll
- Büyük tablolar için sayfalama
- Sonsuz scroll implementasyonu

## Durum Yönetimi

### Loading Durumları

#### Skeleton Loading
```jsx
const SkeletonCard = () => (
    <div className="skeleton-card">
        <div className="skeleton-title" />
        <div className="skeleton-text" />
        <div className="skeleton-text" />
    </div>
);
```

#### Progress Indicators
```css
.progress-bar {
    height: 4px;
    background: var(--primary);
    animation: progress 2s infinite linear;
}

@keyframes progress {
    0% { width: 0; }
    100% { width: 100%; }
}
```

### Error Durumları

#### Error Boundaries
```jsx
class ErrorBoundary extends React.Component {
    state = { hasError: false };

    static getDerivedStateFromError(error) {
        return { hasError: true };
    }

    render() {
        if (this.state.hasError) {
            return <ErrorFallback />;
        }
        return this.props.children;
    }
}
```

#### Error States
- Boş durum gösterimi
- Hata mesajları
- Yeniden deneme seçenekleri

## Dokümantasyon

### Storybook Entegrasyonu

#### Komponent Dokümantasyonu
```jsx
// Button.stories.jsx
export default {
    title: 'Components/Button',
    component: Button,
    argTypes: {
        variant: {
            options: ['primary', 'secondary'],
            control: { type: 'radio' }
        }
    }
};

const Template = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
    variant: 'primary',
    children: 'Primary Button'
};
```

#### Kullanım Örnekleri
- Interaktif demolar
- Prop tabloları
- Kullanım kılavuzları

### Design Tokens

#### CSS Variables
```css
:root {
    /* Colors */
    --primary: #2196F3;
    --secondary: #FF4081;
    --success: #4CAF50;
    --warning: #FFC107;
    --error: #F44336;
    
    /* Typography */
    --font-family: 'Inter', sans-serif;
    --font-size-base: 16px;
    --line-height-base: 1.5;
    
    /* Spacing */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --spacing-lg: 24px;
    --spacing-xl: 32px;
    
    /* Border Radius */
    --border-radius-sm: 4px;
    --border-radius-md: 8px;
    --border-radius-lg: 16px;
    
    /* Shadows */
    --shadow-sm: 0 2px 4px rgba(0,0,0,0.1);
    --shadow-md: 0 4px 8px rgba(0,0,0,0.1);
    --shadow-lg: 0 8px 16px rgba(0,0,0,0.1);
}
```

## Sonuç

Bu UI/UX spesifikasyonu, MedSchedule projesinin kullanıcı arayüzü tasarımını ve kullanıcı deneyimi prensiplerini detaylandırır. Yapay zeka, bu spesifikasyonu kullanarak tutarlı, erişilebilir ve kullanıcı dostu bir arayüz implementasyon etmelidir. Tasarım sistemi, sayfa layoutları, etkileşim tasarımı ve performans optimizasyonları, modern web uygulamaları için en iyi pratikleri takip eder.
