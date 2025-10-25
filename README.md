# LLM-Powered Project Planner

AI destekli proje yönetimi sistemi - Türkçe proje dokümanlarından otomatik Jira task'ları üretir.

---

## 🎯 Sistem Mimarisi

### Aşama 1: Task Planlayıcı (DPO Model)
- **Model**: `ytu-ce-cosmos/Turkish-Llama-8b-DPO-v0.1`
- **Görev**: Proje dokümanını analiz edip Epic ve Task'lara ayırır
- **Çıktı**: Markdown formatında Epic/Task planı (## Epic, ### Task formatında)

### Aşama 2: JSON Dönüştürücü (2 Yöntem)

#### Yöntem 1: MODEL + PARSE (Esnek)
- **Aşama 2a**: Instruct Model markdown'ı JSON'a çevirir
- **Aşama 2b**: Parse fonksiyonu JSON'ı düzeltir (tarih, süre, öncelik formatları)
- **Avantaj**: Karmaşık senaryolarda daha esnek

#### Yöntem 2: DIREKT PARSE (Önerilen ✅)
- **Aşama 2**: Markdown'ı direkt parse ederek JSON'a çevirir
- **Avantaj**: Daha hızlı, daha güvenilir, model hatası riski yok

---

## 🚀 Özellikler

- ✅ **Otomatik Epic Oluşturma**: Projeyi mantıklı iş paketlerine böler
- ✅ **Akıllı Task Atama**: Ekip üyelerinin yeteneklerine göre görev dağıtır
- ✅ **Türkçe Destek**: Tarih, öncelik, süre gibi Türkçe ifadeleri otomatik çevirir
- ✅ **Chunking**: 7+ task için otomatik gruplama (GPU bellek yönetimi)
- ✅ **Jira API Uyumlu**: Direkt Jira'ya gönderilebilir JSON formatı
- ✅ **Ngrok Entegrasyonu**: Cloud ortamdan erişim için public URL

---

## 📦 Kurulum

### Gereksinimler
```bash
pip install torch vllm flask pyngrok
```

### GPU Gereksinimleri
- **CUDA**: 11.8+
- **GPU Memory**: Minimum 16GB (Her model ~7GB)
- **Önerilen**: NVIDIA T4, V100, A100

### Ngrok Kurulumu
```bash
# Ngrok token ayarla
ngrok config add-authtoken YOUR_NGROK_TOKEN
```

---

## 🎮 Kullanım

### Sunucuyu Başlatma
```bash
python llm_powered_project_planner.py
```

Çıktı:
```
✅ Model 1 yüklendi!
✅ Model 2 yüklendi!
📡 Ngrok URL: https://xxxx-xx-xxx.ngrok.io
🔗 API Endpoint: https://xxxx-xx-xxx.ngrok.io/api/generate
💚 Health Check: https://xxxx-xx-xxx.ngrok.io/health
```

### API İsteği
```python
import requests

payload = {
    "json_input": {
        "project_title": "E-Ticaret Platformu",
        "project_description": "Modern bir e-ticaret sistemi geliştirme projesi. Kullanıcı kayıt, ürün yönetimi, sepet ve ödeme modülleri içerecek.",
        "possible_solution": "Mikroservis mimarisi tercih edilmeli. Backend için Django, Frontend için React kullanılabilir.",
        "team": [
            {
                "employee_id": "e14", 
                "name": "Ahmet Yılmaz", 
                "skills": ["python", "django", "postgresql"], 
                "department": "Backend"
            },
            {
                "employee_id": "e07",
                "name": "Ayşe Demir",
                "skills": ["react", "typescript", "css"],
                "department": "Frontend"
            }
        ],
        "metadata": {
            "description": "E-Ticaret platformu MVP geliştirme",
            "company": "TechCorp",
            "department": "Engineering",
            "year": 2025,
            "languages": ["Python", "JavaScript", "TypeScript"]
        }
    },
    "project_key": "ECOM",
    "use_model": False  # True: Model+Parse, False: Direkt Parse (Önerilen)
}

response = requests.post("https://your-ngrok-url/api/generate", json=payload)
result = response.json()

print(f"Toplam Task: {result['total_tasks']}")
print(f"Yöntem: {result['method']}")
```


## 📊 Çıktı Formatı

### Başarılı Yanıt
```json
{
  "success": true,
  "method": "direct_parse",
  "total_tasks": 12,
  "task_plan": "## Epic 1: Backend Altyapısı\n### Task1: API Gateway...",
  "jira_json": {
    "tasks": [
      {
        "epic_name": "Backend Altyapısı",
        "fields": {
          "project": {"key": "ECOM"},
          "issuetype": {"name": "Task"},
          "summary": "API Gateway Kurulumu",
          "description": {
            "type": "doc",
            "version": 1,
            "content": [
              {
                "type": "paragraph",
                "content": [
                  {"type": "text", "text": "API Gateway kurulumu ve konfigürasyonu"}
                ]
              }
            ]
          },
          "assignee": {
            "name": "Ahmet Yılmaz",
            "accountId": "e14"
          },
          "priority": {"name": "High"},
          "duedate": "2025-11-15",
          "labels": ["backend", "api", "infrastructure"],
          "parent": {"key": "BACKEND-ALTYAPISI"},
          "timetracking": {"originalEstimate": "3d"}
        }
      }
    ]
  }
}
```

## 📊 Akıllı Özellikler

### 1. Tarih Parsing
```python
"15 Kasım 2025" → "2025-11-15"
"3 Mart 2025" → "2025-03-03"
```

### 2. Süre Tahmini
```python
"3 gün" → "3d"
"2 hafta" → "10d"
"5 saat" → "5h"
```

### 3. Öncelik Çevirisi
```python
"Yüksek" → "High"
"Orta" → "Medium"
"Düşük" → "Low"
"Çok Yüksek" → "Highest"
```

### 4. Epic Key Oluşturma
```python
"Görev Ekleme ve Analizi" → "GOREV-EKLEME-VE-ANALIZI"
"Backend Altyapısı" → "BACKEND-ALTYAPISI"
```

### 5. Assignee Parsing
```python
"Ali Yılmaz (e01)" → {
    "name": "Ali Yılmaz",
    "accountId": "e01"
}
```

---

## 🔧 Yapılandırma

### GPU Bellek Ayarları
```python
# Model 1: Task Planlayıcı
llm_planner = LLM(
    model="ytu-ce-cosmos/Turkish-Llama-8b-DPO-v0.1",
    gpu_memory_utilization=0.45,  # %45 GPU bellek
    max_model_len=4096
)

# Model 2: JSON Dönüştürücü
llm_converter = LLM(
    model="ytu-ce-cosmos/Turkish-Llama-8b-Instruct-v0.1", //Berkesule/kodllama_sft_cosmosllama_merged
    gpu_memory_utilization=0.45,  # %45 GPU bellek
    max_model_len=4096
)
```

### Chunking Ayarları
```python
# 7'den fazla task varsa otomatik gruplama
chunk_size = 7  # Grup başına task sayısı
```

---

## 📈 Performans

| Metrik | Değer |
|--------|-------|
| **Task Üretme Süresi** | ~3-5 saniye (7 task için) |
| **GPU Bellek Kullanımı** | ~40GB (2 model + uzun girdi) |
| **Maksimum Task Sayısı** | Sınırsız (chunking ile) |


---


---

## 📝 Örnek Kullanım Senaryoları

### 1. Küçük Proje (< 7 Task)
```python
payload = {
    "json_input": {
        "project_title": "Blog Sitesi",
        "project_description": "Basit bir blog platformu",
        "team": [{"employee_id": "e01", "name": "Ali", "skills": ["python"], "department": "Backend"}]
    },
    "project_key": "BLOG",
    "use_model": False
}
```

**Sonuç**: Tek seferde işlenir, ~3 saniye

### 2. Orta Proje (7-20 Task)
```python
payload = {
    "json_input": {
        "project_title": "E-Ticaret MVP",
        "project_description": "Kullanıcı, ürün, sepet, ödeme modülleri",
        "team": [...]
    },
    "project_key": "ECOM",
    "use_model": False
}
```

**Sonuç**: 2-3 gruba bölünür, ~10 saniye

### 3. Büyük Proje (20+ Task)
```python
payload = {
    "json_input": {
        "project_title": "Kurumsal ERP Sistemi",
        "project_description": "İnsan kaynakları, finans, stok, CRM modülleri",
        "team": [...]
    },
    "project_key": "ERP",
    "use_model": False
}
```

**Sonuç**: 4+ gruba bölünür, ~20 saniye
