# Ortak Yazar Öneri Sistemi

Bu proje, akademik makale yazarları arasında ortak çalışma ilişkilerini analiz eden ve gelecekteki olası işbirliklerini tahmin eden bir sistem geliştirmeyi amaçlamaktadır. Proje, yazarların daha önceki işbirliklerine dayalı olarak, hangi yazarlarla gelecekte işbirliği yapmaları gerektiğini önerir.

**İçindekiler:**
- Proje Hakkında
- Özellikler
- Gereksinimler
- Kullanım
- Metodoloji
- Algoritmalar ve Modeller
- Sonuçlar

**Proje Hakkında**

Bu proje, yazarların daha önceki makale işbirliklerine dayalı olarak gelecekteki olası işbirliklerini tahmin eden bir sistem geliştirmektedir. Proje, makaleler ve yazarlar arasındaki ilişkileri analiz ederek, hangi yazarların birlikte çalışabileceğini belirler ve bu yazarları önerir. Proje, Neo4j grafik veritabanını kullanarak ilişkileri depolar ve çeşitli makine öğrenimi algoritmalarını kullanarak tahminler yapar.

**Özellikler**

- *Grafik Veritabanı Kullanımı:* Yazarlar, makaleler ve dergiler arasındaki ilişkileri depolamak için Neo4j kullanır.
- *Veri Analizi ve Görselleştirme:* Pandas ve Matplotlib kullanarak veri analizi ve görselleştirme yapılır.
- *Link Prediction Algoritmaları:* Common Neighbors, Preferential Attachment, ve Total Neighbors gibi link prediction metrikleri kullanılır.
- *Makine Öğrenimi Modelleri:* Random Forest, K-Nearest Neighbors, XGBoost ve Naive Bayes gibi çeşitli makine öğrenimi algoritmaları kullanılır.
- *Kişiselleştirilmiş PageRank:* Belirli yazarlar için en etkili makaleleri bulmak amacıyla kişiselleştirilmiş PageRank algoritması kullanılır.

**Gereksinimler**

- Python 3.8+
- Neo4j
- Pandas
- Matplotlib
- Scikit-learn
- XGBoost
- Py2neo

**Kullanım**

**1. Neo4j Sunucusunu Başlatma:**

Neo4j sunucusunu başlatın ve gerekli şema ve veri yüklemelerini gerçekleştirin.

**2. Veritabanı Ayarlarını Yapma:**

graph = Graph("bolt://localhost:7687", auth=("neo4j", "1234")) satırını kendi Neo4j bağlantı bilgilerinizle güncelleyin.

**3. Verileri Yükleyin ve İşleyin:**

Veritabanına veri yüklemek için aşağıdaki sorguları çalıştırın.

    from py2neo import Graph
    import pandas as pd

    graph = Graph("bolt://localhost:7687", auth=("neo4j", "1234"))

    # Verileri yükleme ve işleme
    query = """
    CALL apoc.periodic.iterate(
    'UNWIND ["dblp-ref-0.json", "dblp-ref-1.json", "dblp-ref-2.json", "dblp-ref-3.json"] AS file
     CALL apoc.load.json("https://github.com/neo4j-contrib/training-v3/raw/master/modules/gds-data-science/supplemental/data/" + file)
     YIELD value WITH value
     return value',
    'MERGE (a:Article {index:value.id})
     SET a += apoc.map.clean(value,["id","authors","references", "venue"],[0])
     WITH a, value.authors as authors, value.references AS citations, value.venue AS venue
     MERGE (v:Venue {name: venue})
     MERGE (a)-[:VENUE]->(v)
     FOREACH(author in authors | 
     MERGE (b:Author{name:author})
     MERGE (a)-[:AUTHOR]->(b))
     FOREACH(citation in citations | 
     MERGE (cited:Article {index:citation})
     MERGE (a)-[:CITED]->(cited))', 
     {batchSize: 1000, iterateList: true});
    """
    graph.run(query).to_data_frame()

  **4. Analiz ve Model Eğitimini Gerçekleştirme:**

  Gerekli analizleri yapma ve modelleri eğitme.

  **5. Öneri Sistemini Kullanma:**

  Eğitilen modelleri kullanarak yazar önerileri alma.

  **Metodoloji**

  - **Veri Yükleme ve İşleme**
    
    1. Veri Yükleme: Makale, yazar ve dergi verileri Neo4j veritabanına yüklenir.
    2. Veri Temizleme: Başlığı olmayan makaleler veritabanından silinir.
    3. Grafik Yapısı: Makaleler, yazarlar ve dergiler arasındaki ilişkiler grafik veritabanında modellenir.

  - **Analiz**
    
    1. Düğüm ve İlişki Sayıları: Grafikteki düğüm ve ilişki sayıları analiz edilir ve görselleştirilir.
    2. Atıf Analizi: Makalelerin atıf sayıları analiz edilir ve dağılımı incelenir.
    3. Yazar Analizi: En çok makale yazan yazarlar ve bu yazarların atıf alan makaleleri analiz edilir.

  - **Link Prediction ve Öneri Sistemi**

    1. Link Prediction Metrikleri: Common Neighbors, Preferential Attachment, ve Total Neighbors metrikleri kullanılır.
    2. Makine Öğrenimi Modelleri: Random Forest, K-Nearest Neighbors, XGBoost ve Naive Bayes algoritmaları kullanılarak link prediction modelleri oluşturulur.
    3. Model Değerlendirme: Modellerin doğruluk, hassasiyet ve geri çağırma (recall) metrikleri ile değerlendirilir.
   
  **Algoritmalar ve Modeller**

  - **Random Forest:** Birden fazla karar ağacının oluşturulduğu ve her bir ağacın sonucunun oylandığı bir topluluk öğrenme yöntemidir.
  - **K-Nearest Neighbors (K-NN):** Veri noktalarının komşuluk ilişkilerine dayalı olarak sınıflandırma yapar.
  - **XGBoost:** Ağaç tabanlı gradyan artırma yöntemini kullanan güçlü bir topluluk öğrenme algoritmasıdır.
  - **Naive Bayes:** Bayes teoremine dayalı, özellikle metin sınıflandırma problemlerinde etkili olan bir algoritmadır.
  - **Personalized PageRank:** Belirli yazarlar için en etkili makaleleri bulmak amacıyla kullanılır.

  **Sonuçlar**

  - **Model Performansı:** Random Forest, K-Nearest Neighbors, XGBoost ve Naive Bayes modelleri karşılaştırıldı. En iyi performans gösteren model XGBoost oldu.
  - **Öneri Sistemi:** Eğitim seti ile test seti ayrılarak, 2006 yılı öncesi eğitim seti ve sonrası test seti olarak belirlendi. Makine öğrenimi modelleri ile başarılı bir şekilde     ortak yazar önerileri yapıldı.




