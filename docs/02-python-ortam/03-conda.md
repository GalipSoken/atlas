# 2.3 conda

## Nedir?
**conda**, hem paket yöneticisi hem de ortam yöneticisi olan bir araçtır. Python paketlerinin yanında Python'un kendisini ve C/C++ ile yazılmış sistem kütüphanelerini (ör. CUDA, GDAL, MKL) de kurabilir.

Dağıtımları:
- **Anaconda:** Yüzlerce bilim paketiyle gelen büyük dağıtım.
- **Miniconda:** Sadece conda ve Python; paketler ihtiyaç oldukça kurulur.
- **Miniforge / mamba:** Topluluk kanalı `conda-forge`'u varsayılan kullanan, daha hızlı alternatif.

## Hangi problemi çözer?
- Farklı Python sürümleri (3.9, 3.11, 3.12) aynı makinede kolayca kullanılır.
- Derlenmesi zor bilim kütüphaneleri hazır gelir; pip'in derleme hatalarıyla uğraşılmaz.
- Windows'ta veri bilimi ortamı kurmak kolaylaşır.

## Nasıl çalışır?
- Ortamlar merkezi bir klasörde tutulur (`~/miniconda3/envs/`), proje klasöründe değil.
- Paketler **kanallardan** (channel) gelir: `defaults`, `conda-forge`.
- Ortam tanımı `environment.yml` dosyasında saklanır.

## Mini örnek
```bash
conda create -n risk_model python=3.11      # ortam oluştur
conda activate risk_model
conda install pandas scikit-learn           # conda ile kur
pip install bir-pypi-paketi                 # conda'da olmayanlar pip ile
conda env list                              # tüm ortamlar
conda env export > environment.yml          # ortamı kaydet
conda env create -f environment.yml         # başka makinede kur
conda deactivate
conda remove -n risk_model --all            # ortamı sil
```

`environment.yml`:
```yaml
name: risk_model
channels:
  - conda-forge
dependencies:
  - python=3.11
  - pandas=2.2
  - scikit-learn=1.5
  - pip
  - pip:
      - lightgbm==4.3.0
```

## Nerede kullanılır?
- Veri bilimi ve ML çalışmaları, Jupyter ortamları
- GPU (CUDA) gerektiren derin öğrenme kurulumları
- Python'un kurulu olmadığı veya eski olduğu kurumsal Windows makineleri

## Benzerleri ve farkları
| | venv + pip | conda | uv |
|---|---|---|---|
| Python sürümü yönetimi | Hayır | Evet | Evet |
| Sistem kütüphaneleri | Hayır | Evet | Hayır |
| Hız | Orta | Yavaş (mamba hızlı) | Çok hızlı |
| Ortam yeri | Proje klasörü | Merkezi klasör | Proje klasörü |
| Docker image boyutu | Küçük | Büyük | Küçük |

**Dikkat:**
- conda ve pip'i karıştırırken önce conda paketlerini, sonra pip paketlerini kurun.
- Anaconda'nın `defaults` kanalı belirli büyüklükteki kurumlar için ücretli lisans gerektirebilir; kurumsal kullanımda lisans durumunu kontrol edin veya `conda-forge` kanalını kullanın.

## İlgili sayfalar
- [2.2 venv ve pip](02-venv-pip.md)
- [2.4 uv ve poetry](04-uv-poetry.md)
