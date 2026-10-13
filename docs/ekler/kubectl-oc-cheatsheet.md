# kubectl / oc Cheat Sheet

Ayrıntılar: [5.14 kubectl](../05-kubernetes/14-kubectl-komutlari.md), [5.18 oc](../05-kubernetes/18-oc-komutlari.md). `oc`, `kubectl` komutlarının hepsini destekler.

## Bağlam ve giriş
| kubectl | oc | Açıklama |
|---|---|---|
| `kubectl config get-contexts` | `oc config get-contexts` | Bağlamlar |
| `kubectl config use-context X` | `oc login --token=... --server=...` | Cluster'a geç / giriş |
| `kubectl config set-context --current --namespace=X` | `oc project X` | Namespace değiştir |
| `kubectl get ns` | `oc projects` | Namespace'ler |
| `kubectl auth whoami` | `oc whoami` | Ben kimim |

## Listeleme ve inceleme
| Komut | Açıklama |
|---|---|
| `kubectl get pods -o wide` | Pod'lar, node ve IP ile |
| `kubectl get all` | Yaygın kaynaklar |
| `kubectl get pods -A -l app=X` | Tüm namespace'lerde label filtresi |
| `kubectl get pods -w` | Canlı izle |
| `kubectl describe pod X` | Ayrıntı + **Events** |
| `kubectl get pod X -o yaml` | Tam tanım |
| `kubectl get events --sort-by=.lastTimestamp` | Son olaylar |
| `kubectl explain deployment.spec.strategy` | Alan dokümantasyonu |
| `kubectl top pods` / `top nodes` | Kaynak kullanımı |

## Oluşturma ve değiştirme
| Komut | Açıklama |
|---|---|
| `kubectl apply -f dosya.yaml` / `-f klasor/` / `-k overlay/` | Uygula (Kustomize: `-k`) |
| `kubectl diff -f dosya.yaml` | Uygulamadan önce farkı gör |
| `kubectl delete -f dosya.yaml` | Sil |
| `kubectl create deploy X --image=Y --dry-run=client -o yaml` | YAML şablonu üret |
| `kubectl create configmap X --from-file=ayar.yaml` | ConfigMap |
| `kubectl create secret generic X --from-literal=K=V` | Secret |
| `kubectl edit deploy X` | Canlı düzenle |
| `kubectl label pod X ortam=test` | Etiket ekle |

## Deployment
| Komut | Açıklama |
|---|---|
| `kubectl scale deploy X --replicas=3` | Ölçekle |
| `kubectl set image deploy/X c=image:2.0` | Image güncelle |
| `kubectl rollout status deploy/X` | İlerleme |
| `kubectl rollout history deploy/X` | Geçmiş |
| `kubectl rollout undo deploy/X` | Geri al |
| `kubectl rollout restart deploy/X` | Pod'ları yenile |
| `kubectl set env deploy/X KEY=val` | Ortam değişkeni |

## Hata ayıklama
| Komut | Açıklama |
|---|---|
| `kubectl logs X` / `-f` / `--previous` | Log / canlı / çöken önceki container |
| `kubectl logs -l app=X --tail=50` | Label'daki tüm pod'ların logu |
| `kubectl exec -it X -- bash` (oc: `oc rsh X`) | İçine gir |
| `kubectl port-forward svc/X 8080:80` | Yerel porta bağla |
| `kubectl cp X:/yol/dosya ./dosya` | Dosya kopyala |
| `kubectl debug -it X --image=busybox` | Hata ayıklama container'ı |
| `kubectl run t --rm -it --image=nicolaka/netshoot -- bash` | Ağ test pod'u |
| `kubectl auth can-i delete pods` | Yetkim var mı |

## Job / CronJob
| Komut | Açıklama |
|---|---|
| `kubectl get cronjobs,jobs` | Liste |
| `kubectl create job --from=cronjob/X elle-1` | Takvimi beklemeden çalıştır |
| `kubectl logs job/elle-1` | Job logu |
| `kubectl patch cronjob X -p '{"spec":{"suspend":true}}'` | Durdur |

## OpenShift'e özgü
| Komut | Açıklama |
|---|---|
| `oc new-project X` | Proje oluştur |
| `oc new-app python:3.11~https://git/repo.git` | Koddan uygulama (S2I) |
| `oc start-build X --from-dir=. --follow` | Build |
| `oc expose svc/X` / `oc create route edge X --service=X` | Route (HTTP / HTTPS) |
| `oc get route X -o jsonpath='{.spec.host}'` | URL |
| `oc debug deploy/X` | Aynı ayarlarla hata ayıklama pod'u |
| `oc adm policy add-role-to-group edit veri-ekibi -n X` | Gruba rol |
| `oc adm groups sync --sync-config=ldap.yaml --confirm` | LDAP grup senkronizasyonu |
| `oc get scc` | Güvenlik kısıtları |

## Pod durumları ve anlamları
| Durum | Muhtemel neden |
|---|---|
| `Pending` | Kaynak yetersiz, PVC bağlanamadı |
| `ImagePullBackOff` | Image adı/etiketi yanlış, registry yetkisi yok |
| `CrashLoopBackOff` | Uygulama açılışta çöküyor → `logs --previous` |
| `OOMKilled` | Bellek limiti aşıldı |
| `CreateContainerConfigError` | Olmayan ConfigMap/Secret |

## Kısaltmalar
`po` pods · `deploy` deployments · `svc` services · `ns` namespaces · `cm` configmaps · `pvc` persistentvolumeclaims · `ing` ingress · `sts` statefulsets · `ds` daemonsets · `cj` cronjobs · `hpa` horizontalpodautoscalers
