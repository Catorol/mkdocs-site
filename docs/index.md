<!-- HEALTHCHECK_TOKEN: mkdocs-helios-ok-2026 -->
# Задание 3. Генераторы статических сайтов для публикации результатов исследований

## ХОД РАБОТЫ

## Ссылки

| Что | URL |
|-----|-----|
| Репозиторий | https://github.com/Catorol/mkdocs-site |
| GitHub Pages | https://catorol.github.io/mkdocs-site/ |
| Helios (main) | https://se.ifmo.ru/~s564527/ |
| Helios preview | https://se.ifmo.ru/~s564527/preview/preview-test/ |
| Actions (успешный run) | https://github.com/Catorol/mkdocs-site/actions/runs/36986390279 |
| Actions (проваленный run) | https://github.com/Catorol/mkdocs-site/actions/runs/36984848631 |

## Лицензии
- Код (CI, конфиги): MIT — файл `LICENSE`
- Контент `docs/`: CC BY 4.0 — файл `LICENSE-CONTENT`

## T3. Обзор отечественных CI/CD-сервисов и статических хостингов

### Платформы CI/CD

| Критерий | GitVerse | SourceCraft | GitFlic |
|---|---|---|---|
| **Модель раннеров** | Облачные (управляются GitVerse) и self-hosted, есть раннеры организации. Облачные без доступа к docker.sock | Облачные воркеры (VM 4 vCPU / 8 ГБ), serverless-воркеры, self-hosted (в том числе в Kubernetes) | В SaaS описаны только свои агенты: Shell, PowerShell, Docker, запуск в Docker-контейнере или Kubernetes. Регистрируются на уровне компании |
| **Синтаксис** | YAML в `.gitverse/workflows/`, контекст `gitverse.*`. Дополнительно читает `.yaml`-файлы из `.github/workflows/` | Свой DSL в `.sourcecraft/ci.yaml`: `on` - `workflows` - `tasks` - `cubes` | `gitflic-ci.yaml`, близок к GitLab CI: `stages`, `rules`, `needs`, `extends`, `include` |
| **Совместимость** | Заявлена совместимость с GitHub Actions | Запуск отдельных GitHub Actions через кубик с ключом `action`, а также пайплайнов с синтаксисом GitLab. Только на облачных воркерах | С GitLab CI: высокая. С GitHub Actions: нет |
| **Каталог готовых действий** | `uses:` из примеров документации: `actions/checkout`, `setup-python`, `setup-node`. Есть локальные действия `./.gitverse/actions/...` | Любые действия из GitHub Marketplace, общедоступные workflow, переиспользуемые кубики | Аналога Marketplace нет. Есть шаблоны конфигураций, компоненты и `include` |
| **Секреты** | Секреты и переменные уровня репозитория и организации, `secrets: inherit` | `${{ secrets.X }}`, Yandex Lockbox, сервисные подключения, дающие IAM-токен без статических ключей | Переменные CI/CD в настройках проекта, HashiCorp Vault, StarVault |
| **Артефакты** | 500 МБ суммарно, хранение 30 дней | 100 МБ на кубик, 10 ГБ на организацию, хранение 14 дней | `artifacts: paths` и `expire_in` |
| **Кэш** | Кэш есть и не входит в лимит артефактов. Подробностей про `actions/cache` не нашла | В справочнике кубиков ключа `cache` нет | `cache: key/paths`, хранится локально на агенте. При нескольких агентах он может быть недоступен |
| **Бесплатный тариф** | 2000 мин CI/CD в месяц, 1 ГБ под артефакты и пакеты, 3 собственных раннера для приватных репозиториев | 1000 мин/мес на организацию, 5 параллельных процессов. Кубик до 1200 с, задание до 3600 с. Self-hosted квоту не расходует | приватные репозитории до 5 человек, репозиторий 4 ГБ, коммит 100 МБ |
| **Встроенный хостинг статики** | GitVerse Pages | SourceCraft Sites | Нет |
| **Документация** | Структурированная, но с расхождениями по лимитам | Подробная, на русском и английском. С 1.10.2026 действуют новые тарифы, старый файл `.src.ci.yaml` признан устаревающим | Большая, на mkdocs, со справочником и примерами. Для SaaS нужно разворачивать агента |

### Варианты размещения

| Вариант | Доставка файлов | Свой домен и HTTPS | Цена и лимиты | Комментарий |
|---|---|---|---|---|
| **Helios ИТМО** | SSH/scp на порт 2222 в `~/public_html` (также SFTP) | Адрес вида `se.ifmo.ru/~номер/`. Для папок нужны права 755. Свой домен невозможен, HTTPS даёт `se.ifmo.ru` | Бесплатно для студентов | Пароль выдаётся через `se.ifmo.ru/passwd`. Для CI пароль или ключ придётся хранить в секретах |
| **Yandex Object Storage** | S3 API (`aws s3 sync`, `yc`) | По умолчанию сайт доступен только по HTTP. Для HTTPS нужен сертификат, его можно взять из Certificate Manager. Домен привязывается через CNAME / Cloud DNS | Первые 100 ГБ исходящего трафика в месяц не тарифицируются. Первый 1 ГБ хранения и 100 000 GET бесплатны | Нужен платёжный аккаунт |
| **VK Cloud** | S3-совместимое хранилище, `aws s3 sync --endpoint-url` | В руководстве сайт на S3 публикуется через CDN | Считается индивидуально через калькулятор цен | Домен и HTTPS через CDN |

Какая доставка чем обслуживается:

- **SSH/scp** подходит для Helios и для любого своего сервера.
- **S3 API** подходит для Yandex, VK Cloud, Selectel, Timeweb и Cloud.ru. Это одна и та же команда с разным `--endpoint-url`.
- **Git push** нужен для GitVerse Pages и SourceCraft Sites.
- **WebDAV и FTP** в документации этих хостингов я как основной способ не нашла. Для S3-хранилищ Selectel упоминаются FTP/SFTP.

### Оценка миграции workflow на отечественные платформы

| Что в workflow | GitVerse | SourceCraft | GitFlic |
|---|---|---|---|
| `name:` | без изменений | станет ключом в `workflows:` (переписать) | имя job (переписать) |
| `on: push: branches: [main]` | без изменений | переписать: `on.push[].workflows` + `filter.branches` | переписать в стиле GitLab (`rules`) |
| `workflow_dispatch` | без изменений | ручной запуск пайплайна | ручной запуск пайплайна |
| `permissions: contents: write` | не нужно (git-пуша нет) | не нужно, если не пушить в ветку; иначе нужен токен записи | не нужно |
| `runs-on: ubuntu-latest` | без изменений | заменить на `image:` в кубике | `image:` плюс собственный агент с тегами |
| `actions/checkout@v4` | без изменений | удалить: репозиторий клонируется сам | удалить |
| `actions/setup-python@v5` | без изменений | кубик `action:` или образ с Python | заменить образом `python:3.12` |
| `pip install "mkdocs==1.6.1" ...` | без изменений (`run:`) | без изменений (`script:`) | без изменений (`script:`) |
| «Configure Git» | удалить | нужен, только если оставить  `gh-deploy` | удалить |
| `mkdocs gh-deploy --force --strict` | **заменить**: `mkdocs build --strict` + upload + deploy | можно оставить при наличии push-доступа, плюс `sites.yaml` | **заменить**: build + доставка из P4 |

## P4. Развёртывание на Helios с контролем качества доставки

### Время доставки и размер

| Шаг | GitHub Pages | Helios |
|---|---|---|
| Set up job | 1 | 2 |
| Checkout | 1 | 1 |
| Setup Python | 0 | 0 |
| Install MkDocs / Install dependencies | 6 | 8 |
| Configure Git | 0 | — |
| Resolve deploy path and site_url | — | 0 |
| Patch site_url for this deploy | — | 0 |
| Build | в составе следующего шага | 0 |
| Build and deploy to gh-pages | 3 | — |
| Setup SSH | — | 4 |
| Backup previous version (rollback point) | — | 3 |
| Rsync deploy | — | 8 |
| Healthcheck | — | 2 |
| Post-шаги и Complete job | 0 | 0 |
| **Итого job** | **14** | **31** |
| **Всего** | **18** | **35** |

| Название | Вес |
|---|---|
| GitHub Pages| 2,35 МБ |
| Helios | 1,99 МБ |

### Надёжность

**GitHub Pages**
- Высокая доступность инфраструктуры GitHub.
- Мало «движущихся частей»: токен встроенный, нет своего SSH.
- Сбой обычно в сборке (`--strict`, зависимости) или в настройке Source (branch vs Actions).

**Helios**
- Зависит от сервера ИТМО (FreeBSD, порт 2222, учетная запись `s564527`).
- Нужны корректные Secrets: ключ, host, port, user, пути `public_html` / `site_releases`.
- Ошибки типичны на стыке Linux-runner <-> FreeBSD (`xargs -r`, пути, `~` в кавычках).
- После выноса бэкапов в `~/site_releases/` исключена рекурсия `_releases` внутри `public_html`.

### Удобство отладки

| | GitHub Pages | Helios |
|--|--------------|--------|
| Логи | только Actions (+ вкладка Pages) | Actions **и** SSH: `ls`, `head`, `curl` на сервере |
| Проверка файла на диске | нет (только через URL) | прямой доступ к `~/public_html/`, `~/site_releases/` |
| Preview по веткам | неудобно из коробки | отдельный подкаталог, сразу видно |
| Откат | revert + новый deploy или ручной rollback branch | `cp -Rp ~/site_releases/root-…/. ~/public_html/` |
| Healthcheck в CI | обычно не делают | обязательный шаг: 200 + контрольная строка |

На Helios отладка богаче: можно отличить «не собралось», «не доехало», «доехало, но неверный `site_url`».

### Preview-сборки

- **main / master** -> корень `public_html` (боевой URL).
- **остальные ветки** -> `public_html/preview/<безопасное-имя-ветки>/`.
- В workflow для каждой ветки подставляется свой `site_url`, иначе ломаются CSS и ссылки в подкаталоге.

На Pages такой схемы «из коробки» нет; для отчёта зафиксировано, что требование preview выполнено на Helios.

### Откат

1. Бэкапы: `~/site_releases/`, например `root-20261002-113947/`, файл `LAST_BACKUP_root.txt`.
2. **До:** сайт с текстом «Тест».
3. Команды: очистка `public_html` кроме `preview` -> `cp -Rp ~/site_releases/root-20261002-113947/. ~/public_html/`.
4. **После:** страница «Welcome to MkDocs».
5. Каталог `preview/` не затрагивался.

![До отката](https://i.ibb.co/jvkNrVwq/image.png)
![Команды](https://i.ibb.co/Xrb9FBKc/1.png)
![Команды](https://i.ibb.co/0y497DfH/image.png)
![После отката](https://i.ibb.co/Mx71F5cY/2.png)

### Поведение при обрыве развёртывания на середине

**GitHub Pages**  
Пока job не завершил publish, пользователи обычно продолжают видеть **предыдущую** успешную версию. Частично опубликованного дерева в привычном виде нет: либо старый сайт, либо (редко) кратковременная неконсистентность на CDN.

**Helios + rsync `--delete`**  
При обрыве SSH/rsync часть файлов уже новая, часть старая или удалённая, возможна **битая** смесь.

Меры в пайплайне:
1. **Сначала backup** текущего дерева в `~/site_releases/`.
2. Затем `rsync`.
3. Затем **healthcheck** (HTTP 200 + `mkdocs-helios-ok-2026`); при неуспехе job падает.
4. Восстановление: копирование из `site_releases` (как при учебном откате).

Обрыв на Helios опаснее, чем на Pages; компенсируется бэкапом до выкладки и проверкой после.

## Ошибки и исправления

**1. Рекурсивный бэкап `_releases` на Helios**  
- **Ошибка:** `cp: .../_releases/.../_releases/...: name too long`; затем `LAST_BACKUP_root.txt: Нет такого файла или каталога`.  
- **Причина:** каталог бэкапов лежал **внутри** `public_html`, backup копировал его в себя; плюс особенности FreeBSD (нет GNU `xargs -r`), неверное раскрытие путей/`~`.  
- **Решение:** бэкапы в `~/site_releases/` **вне** сайта; копирование без `preview`; пути от `$HOME`; совместимые с FreeBSD команды; secret `HELIOS_RELEASES=site_releases`.

**2. Падение step Backup в Actions**  
- **Ошибка:** job красный на «Backup previous version».  
- **Причина:** см. п. 4; redirect в несуществующий каталог после неудачного `cd`.  
- **Решение:** переписан step Backup (heredoc `'ENDSSH'`, `mkdir -p`, запись `LAST_BACKUP` по абсолютному пути).

## Пайплайны

### GitHub Pages

![Успешный деплой](https://i.ibb.co/ycxDzPrj/image.png)

```
# Публикация документации на GitHub Pages через ветку gh-pages.
# Подход: mkdocs gh-deploy (сборка + push в gh-pages), не upload-pages-artifact.
name: Build and Deploy to GitHub Pages

on:
  push:
    branches: [main]          # только основная ветка -> «боевой» сайт Pages
  workflow_dispatch:          # ручной запуск из вкладки Actions

permissions:
  contents: write             # нужно, чтобы bot мог пушить в gh-pages

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # Клонирование репозитория на runner
      - name: Checkout
        uses: actions/checkout@v4

      # Python для MkDocs
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      # Фиксированные версии — воспроизводимая сборка
      - name: Install MkDocs
        run: |
          pip install "mkdocs==1.6.1" "mkdocs-material==9.5.39"

      # Автор коммитов в gh-pages — github-actions[bot]
      - name: Configure Git
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "41898282+github-actions[bot]@users.noreply.github.com"

      # --strict: битые ссылки и предупреждения = ошибка CI
      # --force: перезапись gh-pages
      - name: Build and deploy to gh-pages
        run: mkdocs gh-deploy --force --strict
```

### Helios

![Успешный деплой](https://i.ibb.co/S4pCFjsg/image.png)

```
# Выкладка на Helios ИТМО: SSH-ключ, rsync, backup, healthcheck, preview по веткам.
name: Deploy to Helios

on:
  push:
    branches: ['**']          # main -> корень; остальные -> preview/<branch>/
  workflow_dispatch:

# Один активный deploy на ветку; новый push отменяет предыдущий run
concurrency:
  group: helios-${{ github.ref_name }}
  cancel-in-progress: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          pip install "mkdocs==1.6.1" "mkdocs-material==9.5.39"
          # альтернатива: pip install -r requirements.txt

      # main/master -> public_html; иначе -> public_html/preview/<safe-name>/
      # site_url и public_url для CSS/ссылок и healthcheck
      - name: Resolve deploy path and site_url
        id: meta
        run: |
          BRANCH="${GITHUB_REF_NAME}"
          SAFE=$(echo "$BRANCH" | sed 's#[^a-zA-Z0-9._-]#-#g')
          if [ "$BRANCH" = "main" ] || [ "$BRANCH" = "master" ]; then
            echo "remote_subdir=" >> $GITHUB_OUTPUT
            echo "site_url=${{ secrets.SITE_URL_BASE }}/" >> $GITHUB_OUTPUT
            echo "public_url=${{ secrets.SITE_URL_BASE }}/" >> $GITHUB_OUTPUT
          else
            echo "remote_subdir=preview/${SAFE}" >> $GITHUB_OUTPUT
            echo "site_url=${{ secrets.SITE_URL_BASE }}/preview/${SAFE}/" >> $GITHUB_OUTPUT
            echo "public_url=${{ secrets.SITE_URL_BASE }}/preview/${SAFE}/" >> $GITHUB_OUTPUT
          fi

      # Типичная ошибка: site_url от Pages на Helios в подкаталоге ломает ассеты
      - name: Patch site_url for this deploy
        run: |
          python - << 'PY'
          import pathlib, re, os
          p = pathlib.Path("mkdocs.yml")
          text = p.read_text(encoding="utf-8")
          url = os.environ["SITE_URL"]
          if re.search(r"^site_url:\s*.*$", text, re.M):
              text = re.sub(r"^site_url:\s*.*$", f"site_url: {url}", text, count=1, flags=re.M)
          else:
              text = f"site_url: {url}\n" + text
          p.write_text(text, encoding="utf-8")
          print("site_url ->", url)
          PY
        env:
          SITE_URL: ${{ steps.meta.outputs.site_url }}

      # Строгая сборка — как в требованиях к CI
      - name: Build
        run: mkdocs build --strict

      # Отдельный deploy-ключ (secret HELIOS_SSH_KEY), не пароль аккаунта
      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.HELIOS_SSH_KEY }}" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          ssh-keyscan -p ${{ secrets.HELIOS_PORT }} ${{ secrets.HELIOS_HOST }} >> ~/.ssh/known_hosts

      # Бэкап в ~/site_releases (вне public_html), чтобы не было рекурсии _releases
      # FreeBSD: cp -Rp, без GNU xargs -r; храним 5 последних снимков
      - name: Backup previous version (rollback point)
        run: |
          REMOTE="${{ secrets.HELIOS_USER }}@${{ secrets.HELIOS_HOST }}"
          PORT="${{ secrets.HELIOS_PORT }}"
          BASE="${{ secrets.HELIOS_PATH }}"
          RELEASES="${{ secrets.HELIOS_RELEASES }}"
          SUB="${{ steps.meta.outputs.remote_subdir }}"
          if [ -n "$SUB" ]; then
            TARGET="$BASE/$SUB"
            TAG="prev-$(echo "$SUB" | sed 's#/#-#g')"
          else
            TARGET="$BASE"
            TAG="root"
          fi
          ssh -p "$PORT" -o StrictHostKeyChecking=yes "$REMOTE" \
            "TARGET=$TARGET" "RELEASES=$RELEASES" "TAG=$TAG" \
            sh -s << 'ENDSSH'
          set -e
          cd "$HOME"
          mkdir -p "$TARGET" "$RELEASES"
          STAMP=$(date +%Y%m%d-%H%M%S)
          DEST="$HOME/$RELEASES/${TAG}-$STAMP"
          mkdir -p "$DEST"
          if [ -n "$(ls -A "$HOME/$TARGET" 2>/dev/null)" ]; then
            if [ "$TAG" = "root" ]; then
              for item in "$HOME/$TARGET"/*; do
                [ -e "$item" ] || continue
                name=$(basename "$item")
                [ "$name" = "preview" ] && continue
                [ "$name" = "_releases" ] && continue
                cp -Rp "$item" "$DEST/"
              done
            else
              cp -Rp "$HOME/$TARGET"/. "$DEST/" 2>/dev/null || true
            fi
            echo "BACKUP=${TAG}-$STAMP" > "$HOME/$RELEASES/LAST_BACKUP_${TAG}.txt"
            cd "$HOME/$RELEASES"
            count=0
            for d in $(ls -1dt ${TAG}-* 2>/dev/null); do
              count=$((count + 1))
              if [ "$count" -gt 5 ]; then
                rm -rf "$d"
              fi
            done
            echo "Backup OK: $DEST"
          else
            echo "Target empty, skip backup"
            rmdir "$DEST" 2>/dev/null || true
          fi
          ENDSSH

      # Доставка статики; --delete убирает лишние файлы на сервере
      - name: Rsync deploy
        run: |
          REMOTE="${{ secrets.HELIOS_USER }}@${{ secrets.HELIOS_HOST }}"
          PORT="${{ secrets.HELIOS_PORT }}"
          BASE="${{ secrets.HELIOS_PATH }}"
          SUB="${{ steps.meta.outputs.remote_subdir }}"
          if [ -n "$SUB" ]; then
            TARGET="$BASE/$SUB"
          else
            TARGET="$BASE"
          fi
          ssh -p "$PORT" "$REMOTE" "mkdir -p '$TARGET'"
          rsync -avz --delete \
            -e "ssh -p $PORT" \
            site/ "$REMOTE:$TARGET/"

      # Контроль качества: HTTP 200 + токен из docs/index.md
      - name: Healthcheck
        run: |
          URL="${{ steps.meta.outputs.public_url }}"
          TOKEN="mkdocs-helios-ok-2026"
          echo "Checking $URL"
          for i in 1 2 3 4 5; do
            CODE=$(curl -sS -o /tmp/body.html -w "%{http_code}" "$URL" || echo "000")
            if [ "$CODE" = "200" ] && grep -q "$TOKEN" /tmp/body.html; then
              echo "Healthcheck OK (HTTP $CODE, token found)"
              exit 0
            fi
            echo "Attempt $i: HTTP $CODE, token missing or error — retry..."
            sleep 3
          done
          echo "Healthcheck FAILED"
          echo "HTTP code: $CODE"
          head -c 500 /tmp/body.html || true
          exit 1
```

## Вывод

Для публикации результатов учебных и исследовательских проектов в формате статического сайта рекомендуемый стек:
- генератор: MkDocs + Material;
- CI: GitHub Actions (сборка --strict);
- публичный просмотр: GitHub Pages;
- сдача/вуз: Helios по SSH/rsync + deploy-ключ, healthcheck, preview-ветки, откат из site_releases.

Ограничения, при которых рекомендация меняется:
- только контур РФ / запрет GitHub -> GitVerse/SourceCraft + Object Storage или Helios;
- нужен свой домен и HTTPS без вузовского URL -> Yandex Object Storage + CDN, не Helios;
- команда без доступа к Helios -> только Pages или S3;
- жёсткие требования к preview/rollback в CI -> Helios-схема предпочтительнее «голого» Pages.
