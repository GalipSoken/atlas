# 5.18 oc Komutları

## Nedir?
**oc**, OpenShift'in komut satırı aracıdır. `kubectl`'in tüm komutlarını içerir; üzerine OpenShift'e özgü komutlar (login, project, new-app, route, build) ekler.

Kural: `kubectl get pods` çalışıyorsa `oc get pods` da çalışır.

## Hangi problemi çözer?
OpenShift'e giriş yapmak, Project değiştirmek, uygulama oluşturmak, build almak ve Route yönetmek gibi işleri tek araçla yapar.

## Nasıl çalışır?
- `oc login` ile token alınır ve `~/.kube/config` dosyasına yazılır (kubectl de aynı dosyayı kullanır).
- Web konsolunda sağ üstteki kullanıcı menüsünden **Copy login command** ile hazır login komutu alınabilir.

## Mini örnek

### Giriş ve proje
| Komut | Açıklama |
|---|---|
| `oc login https://api.ocp.sirket.local:6443 -u mehmet` | Kullanıcı adıyla giriş |
| `oc login --token=sha256~... --server=https://...` | Token ile giriş (konsoldan kopyalanır) |
| `oc whoami` | Aktif kullanıcı |
| `oc whoami --show-console` | Web konsolu adresi |
| `oc projects` | Erişilebilen projeler |
| `oc project veri-dev` | Projeye geç |
| `oc new-project veri-dev` | Yeni proje |
| `oc status` | Projedeki uygulamaların özeti |

### Uygulama oluşturma
| Komut | Açıklama |
|---|---|
| `oc new-app python:3.11~https://git.../repo.git` | S2I ile koddan uygulama |
| `oc new-app --docker-image=registry.../app:1.0` | Hazır image'dan |
| `oc new-app -f template.yaml` | Şablondan |
| `oc expose svc/app` | Route oluştur |
| `oc create route edge app --service=app` | HTTPS route |
| `oc get route` | URL'ler |

### Build
| Komut | Açıklama |
|---|---|
| `oc get bc` | BuildConfig'ler |
| `oc start-build app --follow` | Build başlat ve izle |
| `oc start-build app --from-dir=. --follow` | Yerel klasörden build |
| `oc logs -f bc/app` | Son build'in logu |
| `oc cancel-build app-3` | Build iptal |

### Çalışma zamanı ve hata ayıklama
| Komut | Açıklama |
|---|---|
| `oc get pods` | Pod'lar |
| `oc logs -f deploy/app` | Loglar |
| `oc rsh pod-adi` | Pod'da shell aç (kubectl exec -it ... sh) |
| `oc rsync ./yerel pod-adi:/app` | Dosya senkronize et |
| `oc port-forward pod-adi 8080:8080` | Yerel porta bağla |
| `oc debug deploy/app` | Aynı ayarlarla hata ayıklama pod'u aç |
| `oc debug deploy/app --as-root` | Root olarak (yetki gerekir) |
| `oc set env deploy/app LOG_LEVEL=DEBUG` | Ortam değişkeni ekle |
| `oc set env deploy/app --from=secret/db-secret` | Secret'tan değişken |
| `oc set resources deploy/app --limits=memory=1Gi` | Kaynak sınırı |
| `oc scale deploy/app --replicas=3` | Ölçekleme |
| `oc rollout restart deploy/app` | Yeniden başlat |
| `oc rollout undo deploy/app` | Geri al |

### Yetki ve yönetim
| Komut | Açıklama |
|---|---|
| `oc adm policy add-role-to-user edit mehmet -n veri-dev` | Kullanıcıya rol |
| `oc adm policy add-role-to-group view veri-ekibi -n veri-prod` | Gruba rol |
| `oc get rolebindings` | Yetkiler |
| `oc adm groups sync --sync-config=ldap-sync.yaml --confirm` | LDAP gruplarını senkronize et (yönetici) |
| `oc adm top pods` | Kaynak kullanımı |
| `oc get events --sort-by=.lastTimestamp` | Son olaylar |

## Nerede kullanılır?
Tipik geliştirici akışı:
```bash
oc login --token=... --server=https://api.ocp.sirket.local:6443
oc project veri-dev
oc start-build kredi-api --from-dir=. --follow
oc rollout status deploy/kredi-api
oc logs -f deploy/kredi-api
oc get route kredi-api -o jsonpath='{.spec.host}'
```

## Benzerleri ve farkları
| İş | kubectl | oc |
|---|---|---|
| Giriş | kubeconfig elle / bulut CLI | `oc login` |
| Namespace değiştirme | `kubectl config set-context --current --namespace=X` | `oc project X` |
| Pod'a shell | `kubectl exec -it X -- sh` | `oc rsh X` |
| Dışa açma | Ingress YAML | `oc expose svc/X` |
| Koddan uygulama | — | `oc new-app` |

- **Web konsolu:** Aynı işlerin çoğu konsoldan da yapılabilir; "Developer" görünümü uygulama topolojisini görsel gösterir.
- **odo:** Geliştiricilere yönelik, OpenShift'te hızlı geliştirme döngüsü sunan ayrı bir CLI.

## İlgili sayfalar
- [5.14 kubectl komutları](14-kubectl-komutlari.md)
- [5.17 OpenShift kavramları](17-openshift-kavramlari.md)
