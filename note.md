# NYC Taksi Səfərləri: Saatlıq və Gündəlik Tələbat Analizi

## 1. Dataset və alətlər

- **Dataset:** Kaggle, *NYC Taxi Trip Duration* (`train.csv`). Tapşırıqda təklif olunan dataset istifadə olunub, başqa dataset götürülməyib.
- **Həcm:** 1 458 644 sətir, 11 sütun. Heç bir sütunda boş (missing) dəyər yoxdur.
- **Əhatə etdiyi dövr:** 01.01.2016 – 30.06.2016 (cəmi 6 ay).
- **Alətlər:** Analiz **R dilində** (RStudio, R Markdown) aparılıb. 
- **Fayllar:** `NYC_taxi_analysis.Rmd` (kod və şərhlər), `NYC_taxi_analysis.html` (nəticələr), `figures/` qovluğu (qrafiklər).

## 2. Metodologiya

1. `pickup_datetime` sütunu `ymd_hms()` ilə datetime tipinə çevrildi.
2. `trip_duration` saniyədən dəqiqəyə çevrildi (60-a bölünərək, `trip_duration_min`).
3. **Outlier filtri:** 1 dəqiqədən qısa (8 595 səfər) və 120 dəqiqədən uzun (2 253 səfər) səfərlər silindi. Cəmi **10 848 sətir (0,74%)** silindi, **1 447 796 sətir** qaldı.
4. `pickup_datetime`-dan saat (`hour`), həftənin günü (`weekday`), ay (`month`) və gün tipi (`day_type`: iş günü / həftə sonu) çıxarıldı.
5. Saat, gün və ay üzrə səfər sayı `count()` ilə hesablandı.
6. İş günü ilə həftə sonu müqayisəsi **gündəlik orta səfər sayı** ilə aparıldı, çünki 5 iş günü ilə 2 həftə sonu günündəki ümumi səfər sayını birbaşa müqayisə etmək düzgün deyil.
7. Hər saat üçün orta və median səfər müddəti hesablandı.
8. Bonus: saat × gün heatmap-ı və `vendor_id=1` ilə `vendor_id=2` müqayisəsi (Welch t-test daxil olmaqla).

## 3. Outlier-lərin təsiri

| Göstərici | Filtrdən əvvəl | Filtrdən sonra |
|---|---|---|
| Orta müddət (dəq) | 15,99 | 14,01 |
| Median müddət (dəq) | 11,03 | 11,08 |
| Maksimum müddət (dəq) | 58 771,4 | 120 |

Maksimum dəyər 58 771 dəqiqə (təxminən 40 gün) idi, bu isə açıq-aydın data xətasıdır. Outlier-lər ortalamanı təxminən 2 dəqiqə şişirdirdi, median isə demək olar dəyişmədi. Ona görə sağa əyilmiş paylanmada median daha etibarlı göstəricidir.

Filtrdən sonra müddətin statistikası: orta 14,01 dəq, median 11,08 dəq, standart sapma 10,89 dəq, 25% kvantil 6,68 dəq, 75% kvantil 17,93 dəq. Paylanma sağa əyilib: səfərlərin çoxu qısadır, az sayda uzun səfər ortalamanı medianın üstünə çəkir.

## 4. Əsas tapıntılar

### 4.1 Saat üzrə tələbat (Şəkil 01)

| | Saat | Səfər sayı |
|---|---|---|
| Ən yüksək 3 saat | 18:00 | 90 022 |
| | 19:00 | 89 753 |
| | 21:00 | 83 607 |
| Ən aşağı 3 saat | 05:00 | 14 767 |
| | 04:00 | 15 524 |
| | 03:00 | 20 637 |

Zirvə axşam saatlarındadır (18:00–19:00). Ən sakit saat 05:00-dır. Zirvə saatı ilə ən sakit saat arasında təxminən **6 dəfə** fərq var.

### 4.2 Həftənin günü üzrə tələbat (Şəkil 02)

| Gün | Səfər sayı |
|---|---|
| Bazar ertəsi | 186 051 |
| Çərşənbə axşamı | 201 318 |
| Çərşənbə | 208 751 |
| Cümə axşamı | 216 947 |
| **Cümə** | **221 870** |
| Şənbə | 219 190 |
| Bazar | 193 669 |

Ən çox səfər **Cümə**, ən az səfər **Bazar ertəsi** günü olub. Tələbat həftənin əvvəlindən Cüməyə qədər artır, Şənbədə yüksək qalır və Bazar günü yenidən düşür.

### 4.3 İş günü və həftə sonu (Şəkil 08)

- Gündəlik orta səfər sayı: **iş günü 7 961, həftə sonu 7 940**. Fərq cəmi təxminən 0,3%-dir, yəni ümumi tələbat demək olar eynidir.
- Saatlıq profil isə tamamilə fərqlidir (qrafikdən oxunub):
  - **İş günləri:** iki zirvə var. Səhər zirvəsi 08:00 ətrafında, axşam zirvəsi 18:00–19:00 ətrafındadır.
  - **Həftə sonu:** səhər zirvəsi yoxdur, 05:00–08:00 arası çox sakitdir. Gündüz tələbat nisbətən düz gedir (12:00–18:00), gecə saatlarında (00:00–03:00) isə həftə sonunun 2 günlük cəmi belə iş günlərinin 5 günlük cəmindən çox və ya ona yaxındır.

### 4.4 Heatmap (Şəkil 06)

- Ən intensiv xanalar iş günü axşamlarındadır: Cümə axşamı 21:00 və Cümə 19:00 xüsusilə yüksəkdir.
- Cümə və Şənbə gecələri 23:00 – 01:00 arası yüksək tələbat var (Şənbə və Bazar günü 00:00 xanaları açıq rəngdədir).
- Bazar ertəsindən Cümə axşamına qədər 03:00–05:00 arası ən aşağı tələbat zonasıdır.
- Şənbə və Bazar günü səhər saatları (05:00–09:00) iş günlərinə nisbətən çox zəifdir.

### 4.5 Saat üzrə orta səfər müddəti (Şəkil 05)

- Ən uzun orta müddət təxminən **15:00–16:00** saatlarındadır (təxminən 16,2 dəq).
- Ən qısa orta müddət **06:00** saatındadır (təxminən 11,3 dəq; median 7,9 dəq).
- Axşam zirvəsində (18:00) orta müddət təxminən 14,4 dəq, 19:00–20:00 arası təxminən 13 dəq-ə düşür.
- Yəni ən çox səfərin olduğu saatlar (18:00–19:00) ən uzun səfərlərin olduğu saatlar deyil.

Qeyd: 15:00–16:00 və axşam saatlarının dəyərləri qrafikdən təxmini oxunub, 24 saatın dəqiq cədvəli Rmd/HTML faylındadır (`avg_dur_hour`).

### 4.6 Aylıq tendensiya (Şəkil 04)

| Ay | Səfər sayı | Orta müddət (dəq) |
|---|---|---|
| Yanvar | 228 064 | 13,2 |
| Fevral | 236 609 | 13,2 |
| Mart | 254 306 | 13,7 |
| Aprel | 249 713 | 14,2 |
| May | 246 579 | 14,8 |
| İyun | 232 525 | 14,9 |

- Ən çox səfər **Mart** ayında, ən az səfər **Yanvar** ayındadır. Mart Yanvardan təxminən 11,5% çoxdur.
- Mart-dan sonra səfər sayı azalır, İyunda Martla müqayisədə təxminən 8,6% aşağıdır.
- Orta səfər müddəti isə Yanvardan İyuna **ardıcıl artır** (13,2 → 14,9 dəq, təxminən 13%). Ən yüksək orta müddət **İyun** ayındadır.
- Bu dəyişikliyin səbəbi (məsələn, hava şəraiti və ya turist mövsümü) datada yoxdur, bu yalnız mümkün izah ola bilər və bu analizdə sübut olunmayıb.

### 4.7 Səfər müddətinin paylanması (Şəkil 03)

Paylanma sağa əyilib: ən çox səfər 5–15 dəqiqə aralığındadır, 120 dəqiqəyə doğru sayı kəskin azalır. Səfərlərin yarısı 11 dəqiqədən qısadır (median 11,08 dəq).

### 4.8 Bonus: Vendor müqayisəsi (Şəkil 07)

| vendor_id | Səfər sayı | Pay | Orta müddət (dəq) | Median (dəq) |
|---|---|---|---|---|
| 1 | 673 164 | 46,5% | 13,93 | 11,0 |
| 2 | 774 632 | 53,5% | 14,08 | 11,1 |

- `vendor_id` sürücü deyil, provayder (şirkət) kodudur. Səfərlərin vendor üzrə bölgüsü təxminən 46,5% / 53,5%-dir.
- Welch t-test nəticəsi: p-value < 2,2e-16, orta müddət fərqi üçün 95% etibarlılıq intervalı: −0,19 ilə −0,12 dəq arasında.
- Fərq statistik cəhətdən əhəmiyyətlidir, amma **praktiki cəhətdən çox kiçikdir** (təxminən 9 saniyə). 1,4 milyon sətirlik datada çox kiçik fərq də əhəmiyyətli çıxır. Ona görə vendorlar arasında real fərq yoxdur kimi qəbul etmək olar.

## 5. Taksi parkı idarəetməsi üçün 4 əməliyyat insight-ı

**1. Maşın sayını zirvə saatlarına uyğunlaşdırmaq.** 18:00–19:00 saatlarında 6 ay ərzində hər saat üçün təxminən 90 min səfər olub, 03:00–05:00 saatlarında isə cəmi 14,8–20,6 min. Zirvə ilə ən sakit saat arasında təxminən 6 dəfə fərq var. *Qərar:* xəttə çıxan maşın sayı 17:00–22:00 arasında maksimuma çatdırılmalı, 03:00–05:00 arası minimuma endirilməlidir. Bu pəncərə texniki baxış, təmizlik və növbə dəyişimi üçün uyğundur.

**2. İş günü və həftə sonu üçün fərqli növbə cədvəli.** Gündəlik orta səfər sayı eynidir (7 961 və 7 940), ona görə həftə sonu parkı ümumi olaraq azaltmaq olmaz. Amma tələbatın forması fərqlidir: iş günləri iki zirvə var (08:00 və 18:00–19:00), həftə sonu səhər zirvəsi yoxdur və gecə tələbatı (Cümə və Şənbə, 23:00–01:00) yüksəkdir. *Qərar:* iş günləri səhər (07:00–09:00) və axşam növbələrini gücləndirmək, həftə sonu isə səhər saatlarında maşınları azaltmaq, gecə saatlarında artırmaq.

**3. Gündüz tıxac saatlarında planlaşdırmanı dəyişmək.** Orta səfər müddəti təxminən 15:00–16:00-da ən yüksəkdir (təxminən 16,2 dəq), 06:00-da isə ən aşağıdır (təxminən 11,3 dəq), yəni təxminən 40% fərq var. Uzun səfərlər o deməkdir ki, hər maşın həmin saatlarda daha az səfər edir. *Qərar:* 14:00–17:00 arası üçün əlavə maşın ehtiyatı ayırmaq, növbə dəyişimini bu saatlara salmamaq və dispetçerlərin tıxaclı marşrutlardan qaçmasını təmin etmək. Axşam zirvəsində (18:00–19:00) səfərlər nisbətən qısadır, bu da maşının daha çox səfər edə bilməsi deməkdir.

**4. Aylıq tələbata görə resursların planlaşdırılması.** Mart ayı ən yüksək (254 306), Yanvar ən aşağı (228 064) aydır, fərq təxminən 11,5%-dir. İyunda səfər sayı Marta nisbətən təxminən 8,6% azalıb, amma orta müddət 13,2 dəq-dən 14,9 dəq-ə qədər artıb. *Qərar:* Mart–May dövründə maşın ehtiyatı artırılmalı, Yanvar–Fevralda texniki xidmət və planlı təmir üçün istifadə edilməlidir. İyunda səfərlər azalsa da uzun olur, buna görə maşın başına gəliri ayrıca izləmək lazımdır.

## 6. Məhdudiyyətlər

- Data yalnız **6 aylıqdır** (yanvar–iyun 2016). Tam illik mövsümilik haqqında nəticə çıxarmaq olmaz, aylıq tendensiya yalnız bu dövrə aiddir.
- İş günü / həftə sonu saatlıq qrafik (Şəkil 08) **ümumi səfər sayını** göstərir, gündəlik orta deyil. İş günlərinin sayı daha çox olduğu üçün dəqiq müqayisədə həftə sonu gecə tələbatı qrafikdə göründüyündən də yüksəkdir.
- Həftənin günləri üzrə say bütün dövrdə hər günün neçə dəfə təkrarlanmasına görə bir qədər fərqlənə bilər, bu analizdə bunun üçün normallaşdırma aparılmayıb.
- Outlier həddi (1 və 120 dəqiqə) tapşırığın qaydasıdır. Başqa hədd seçilərsə, orta müddət dəyişə bilər.
- Datada hava, bayram və ya hadisə məlumatı yoxdur, ona görə tələbat dəyişikliklərinin səbəbləri haqqında yalnız fərziyyə söyləmək olar.
- Saat üzrə orta müddət haqqında bəzi rəqəmlər qrafikdən təxmini oxunub (4.5 bölməsində qeyd olunub).
- Analiz yalnız yola çıxma (pickup) vaxtına əsaslanır, məkan (koordinat) məlumatları istifadə olunmayıb.

## 7. Qrafiklər (`figures/` qovluğu)

| Fayl | Məzmun |
|---|---|
| `01_trips_by_hour.png` | Saat üzrə səfər sayı (line chart) |
| `02_trips_by_weekday.png` | Həftənin günü üzrə səfər sayı (bar chart) |
| `03_duration_histogram.png` | Səfər müddətinin histoqramı |
| `04_monthly_trips.png` | Aylıq səfər sayı (bar chart) |
| `05_avg_duration_by_hour.png` | Saat üzrə orta səfər müddəti |
| `06_heatmap_hour_weekday.png` | Bonus: saat × gün heatmap |
| `07_vendor_duration.png` | Bonus: vendor üzrə orta müddət |
| `08_hourly_weekday_vs_weekend.png` | İş günü və həftə sonu saatlıq tələbat |
