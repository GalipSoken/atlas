# 5.14 kubectl Komutları

## Nedir?
**kubectl**, Kubernetes API'si ile konuşan komut satırı aracıdır. Cluster'daki her nesneyi listelemek, oluşturmak, değiştirmek, silmek ve hata ayıklamak için kullanılır. OpenShift'teki karşılığı `oc`'dir ve kubectl komutlarının tamamını destekler ([5.18](18-oc-komutlari.md)).

## Hangi problemi çözer?
Kubernetes'le günlük çalışmanın neredeyse tamamı birkaç düzine kubectl komutuyla yapılır. Bu sayfa hızlı başvuru içindir.

## Nasıl çalışır?
- Bağlantı bilgileri `~/.kube/config` dosyasındadır (**kubeconfig**).
- Bu dosyada birden fazla **context** (cluster + kullanıcı + varsayılan namespace üçlüsü) olabilir.
- Komut yapısı: `kubectl <fiil> <kaynak türü> <isim> [seçenekler]`

```
kubectl  get   pods   kredi-api-7f9c  -n veri-prod  -o yaml
         fiil  tür    isim            namespace     çıktı biçimi
```

## Mini örnek

### Bağlam ve namespace
| Komut | Açıklama |
|---|---|
| `kubectl config get-contexts` | Tanımlı cluster bağlamları |
| `kubectl config use-context prod` | Bağlam değiştir |
| `kubectl config set-context --current --namespace=veri-dev` | Varsayılan namespace |
| `kubectl get ns` | Namespace'ler |

### Okuma
| Komut | Açıklama |
|---|---|
| `kubectl get pods` | Pod'lar |
| `kubectl get pods -o wide` | Node ve IP bilgisiyle |
| `kubectl get pods -A` | Tüm namespace'ler |
| `kubectl get pods -l app=kredi-api` | Label ile filtre |
| `kubectl get pods -w` | Değişiklikleri canlı izle |
| `kubectl get deploy,svc,ingress` | Birden çok tür |
| `kubectl get all` | Yaygın türlerin hepsi |
| `kubectl get pod X -o yaml` | Tam tanım |
| `kubectl describe pod X` | Detay + **Events** (hata ayıklamada ilk adım) |
| `kubectl get events --sort-by=.lastTimestamp` | Son olaylar |
| `kubectl explain deployment.spec` | Alan dokümantasyonu |

### Oluşturma ve değiştirme
| Komut | Açıklama |
|---|---|
| `kubectl apply -f dosya.yaml` | Tanımı uygula (oluştur veya güncelle) |
| `kubectl apply -f klasor/` | Klasördeki tüm YAML'lar |
| `kubectl diff -f dosya.yaml` | Uygulamadan önce farkı gör |
| `kubectl delete -f dosya.yaml` | Tanımdakileri sil |
| `kubectl delete pod X` | Pod sil (Deployment yenisini açar) |
| `kubectl edit deploy X` | Canlı nesneyi editörde düzenle |
| `kubectl create deploy X --image=nginx --dry-run=client -o yaml > d.yaml` | YAML şablonu üret |

### Deployment işlemleri
| Komut | Açıklama |
|---|---|
| `kubectl scale deploy X --replicas=5` | Kopya sayısı |
| `kubectl set image deploy/X app=image:2.0` | Image güncelle |
| `kubectl rollout status deploy/X` | Güncelleme ilerlemesi |
| `kubectl rollout history deploy/X` | Sürüm geçmişi |
| `kubectl rollout undo deploy/X` | Geri al |
| `kubectl rollout restart deploy/X` | Pod'ları sırayla yenile |

### Hata ayıklama
| Komut | Açıklama |
|---|---|
| `kubectl logs X` | Pod logları |
| `kubectl logs X -c sidecar` | Belirli container |
| `kubectl logs X --previous` | Çöken önceki container'ın logu |
| `kubectl logs -f deploy/X` | Deployment'ın bir pod'unu canlı izle |
| `kubectl exec -it X -- bash` | Pod'un içine gir |
| `kubectl port-forward svc/X 8080:80` | Yerel porta bağla |
| `kubectl cp X:/app/out.csv ./out.csv` | Dosya kopyala |
| `kubectl top pods` / `top nodes` | Kaynak kullanımı |
| `kubectl debug -it X --image=busybox` | Geçici hata ayıklama container'ı |
| `kubectl auth can-i create deploy` | Yetkim var mı? |

## Nerede kullanılır?
**"Pod çalışmıyor" kontrol listesi**
```bash
kubectl get pods                         # STATUS ne? Pending / CrashLoopBackOff / ImagePullBackOff
kubectl describe pod X                   # en alttaki Events bölümü
kubectl logs X --previous                # çökmeden önceki son loglar
kubectl get events --sort-by=.lastTimestamp | tail -20
```

| Durum | Muhtemel neden |
|---|---|
| `Pending` | Yetersiz kaynak, PVC bağlanamadı, node seçici uymuyor |
| `ImagePullBackOff` | Image adı/etiketi yanlış, registry yetkisi yok |
| `CrashLoopBackOff` | Uygulama açılır açılmaz hata veriyor (log'a bakın) |
| `OOMKilled` | Bellek limiti aşıldı |
| `CreateContainerConfigError` | Olmayan ConfigMap/Secret'a referans |

## Benzerleri ve farkları
- **`apply` ile `create`:** `create` sadece oluşturur, varsa hata verir. `apply` deklaratiftir; tekrar tekrar çalıştırılabilir. Tercih edilen `apply`.
- **Kısaltmalar:** `po` (pods), `deploy`, `svc`, `ns`, `cm` (configmap), `pvc`, `ing` (ingress).
- **Yardımcı araçlar:** `k9s` (terminal arayüzü), `kubectx`/`kubens` (hızlı bağlam/namespace değişimi), Lens (masaüstü arayüz).
- **Takma ad:** `alias k=kubectl` yaygın bir alışkanlıktır.

## İlgili sayfalar
- [5.18 oc komutları](18-oc-komutlari.md)
- [kubectl / oc cheat sheet](../ekler/kubectl-oc-cheatsheet.md)
- [17.6 YAML](../17-formatlar/06-yaml.md)
