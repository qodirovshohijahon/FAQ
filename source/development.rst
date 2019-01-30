.. Fedora-Faq-Ru (c) 2018 - 2019, EasyCoding Team and contributors
.. 
.. Fedora-Faq-Ru is licensed under a
.. Creative Commons Attribution-ShareAlike 4.0 International License.
.. 
.. You should have received a copy of the license along with this
.. work. If not, see <https://creativecommons.org/licenses/by-sa/4.0/>.
.. _development:

*****************************************************
Вопросы, связанные с разработкой и сборкой пакетов
*****************************************************

.. index:: repository, package, packaging, rpm
.. _create-package:

Я хочу создать пакет для Fedora. Что мне следует знать?
============================================================

См. `здесь <https://docs.fedoraproject.org/quick-docs/en-US/creating-rpm-packages.html>`__ и `здесь <https://www.easycoding.org/2018/06/17/pravilno-paketim-po-dlya-linux.html>`__.

.. index:: package, packaging, rpm, building
.. _build-package:

Как собрать RPM пакет в mock?
==================================

См. `здесь <https://www.easycoding.org/2017/02/22/sobiraem-rpm-pakety-dlya-fedora-v-mock.html>`__.

.. index:: packaging, repository, maintainer
.. _becoming-maintainer:

Как добавить свой пакет в репозиторий Fedora и стать мейнтейнером?
=====================================================================

См. `здесь <https://www.easycoding.org/2016/06/20/dobavlyaem-paket-v-glavnyj-repozitorij-fedora.html>`__.

.. index:: koji, about
.. _koji-about:

Что такое Koji?
===================

`Fedora Koji <https://koji.fedoraproject.org/koji/>`__ - это автоматизированная среда для сборки пакетов для Fedora.

.. index:: fedpkg, package, rebuild, mock, rpm
.. _fedpkg-rebuild:

Хочу внести свои правки в пакет и пересобрать его для личных нужд. Как проще это сделать?
===============================================================================================

Установим утилиты fedpkg и mock:

.. code-block:: bash

    sudo dnf install fedpkg mock

Скачаем исходники необходимого пакета **foo-bar**:

.. code-block:: bash

    fedpkg clone -a foo-bar

Перейдём в каталог с загруженными исходниками и переключимся на ветку для конкретной версии Fedora (если нужна версия из Rawhide - следует использовать **master**):

.. code-block:: bash

    cd foo-bar
    fedpkg switch-branch f29

Внесём свои правки, сделаем коммит в репозиторий:

.. code-block:: bash

    git add -A
    git commit -m "Description of our changes."

Запустим автоматическую :ref:`сборку в mock <build-package>`:

.. code-block:: bash

    fedpkg mockbuild

.. index:: git, tarball
.. _git-tarball:

Как создать tarball с исходниками из Git репозитория?
=========================================================

Если проект по какой-либо причине не поставляет готовые тарболы и отсутствует возможность их скачать напрямую с хостинга VCS, можно создать их из Git.

Клонируем репозиторий источника:

.. code-block:: bash

    git clone https://example.org/foo-bar.git

Создадим архив с исходниками:

.. code-block:: bash

    git archive --format=tar --prefix=foo-bar-1.0.0/ HEAD | gzip > ~/rpmbuild/SOURCES/foo-bar-1.0.0.tar.gz

Здесь **HEAD** - указатель на актуальный коммит (вместо этого можно использовать SHA1 хеш любого коммита, а также имя тега или ветки), **foo-bar** - название проекта, а **1.0.0** - его версия.

.. index:: fedpkg, koji
.. _rpmfusion-override:

Как переопределить пакет в Koji репозитория RPM Fusion?
===========================================================

Создание build override для репозитория f29-free:

.. code-block:: bash

    koji-rpmfusion tag f29-free-override foo-bar-1.0-1.fc29

Удаление build override для репозитория f29-free:

.. code-block:: bash

    koji-rpmfusion untag f29-free-override foo-bar-1.0-1.fc29

.. index:: git, gmail, mail
.. _git-gmail:

Как настроить Git для работы с почтовым сервисом Gmail?
===========================================================

Для того, чтобы использовать функцию **git send-mail** с почтовым сервисом Gmail, необходимо:

 1. включить двухфакторную аутентификацию в настройках Google аккаунта;
 2. в настройках безопасности почтового ящика Gmail разрешить использование "небезопасных приложений" (под небезопасными Google понимает любые, не поддерживающие OAuth2);
 3. там же включить доступ к почте посредством POP3 или IMAP (это активирует также и необходимый для нас протокол SMTP);
 4. в настройках безопасности сгенерировать новый пароль для приложения;
 5. указать в файле **~/.gitconfig** параметры почтового сервиса;
 6. когда будет запрошен пароль, ввести созданный ранее пароль приложения.

Пример файла **~/.gitconfig** для работы с почтовым сервисом Gmail:

.. code-block:: ini

    [sendemail]
        smtpEncryption = tls
        smtpServer = smtp.gmail.com
        smtpUser = yourname@gmail.com
        smtpServerPort = 587

.. index:: library, shared library, linker, dlopen
.. _dlopen-usage:

Правильно ли использовать dlopen для загрузки динамических библиотек в приложении?
======================================================================================

Для загрузки динамических библиотек в приложении использовать dlopen допускается, но мы настоятельно рекомендуем избегать этого и использовать полноценную линковку по следующим причинам:

 1. в каждом дистрибутиве GNU/Linux именование библиотек, особенно если у них нет чётко установленной апстримом SOVERSION константы, ложится на плечи мейнтейнеров. К примеру есть популярная libcurl. Во всех дистрибутивах она линкуется с openssl и называется libcurl.so, а в Debian и Ubuntu была переименована в libcurl-gnutls.so из-за линковки с gnutls;
 2. нет никакой гарантии, что загрузится именно необходимая версия библиотеки, имеющая необходимую функцию, а отсутствии оной приложение будет аварийно завершено с ошибкой сегментирования;
 3. если существует несколько версий библиотеки с разными SOVERSION, необходимо самостоятельно их искать на диске и подгружать с рядом хаков ибо имя libfoo.so без указанной SOVERSION в большинстве дистрибутивов представляет собой символическую ссылку и доступен лишь после установки соответствующего development пакета. Соответственно на машинах обычных пользователей он отсутствует;
 4. о библиотеках, подгружаемых динамически, не в курсе LD, а следовательно он не сможет при загрузке образа приложения подгрузить их в память;
 5. в случае корректной линковки LD перед запуском приложения осуществит автоматический поиск необходимых экспортируемых функций во всех указанных библиотеках. При их отсутствии приложение не будет запущено;
 6. при сборке пакета динамически подгружаемые через dlopen библиотеки не будут определены и прописаны в качестве зависимостей пакета, что может вызвать проблемы у пользователей и падение приложения;

.. index:: environment, options, env, terminal
.. _env-get-term:

Как получить полный список установленных переменных окружения в текущем терминале?
======================================================================================

Получить список установленных переменных окружения можно посредством выполнения утилиты **env**:

.. code-block:: bash

    env

.. index:: environment, options, env, application
.. _env-get-app:

Как получить полный список установленных переменных для запущенного процесса?
================================================================================

Получение списка установленных переменных окружения для запущенных процессов:

.. code-block:: bash

    cat /proc/$PID/environ

Здесь **$PID** - :ref:`PID <get-pid>` процесса, информацию о котором необходимо получить.

.. index:: environment, options, env
.. _env-set:

Как задать переменную окружения?
====================================

Вариант 1. Запуск процесса с заданной переменной окружения:

.. code-block:: bash

    FOO=BAR /usr/bin/foo-bar

Вариант 2. Экспорт переменной окружения в запущенном терминале и дальнейший запуск приложения:

.. code-block:: bash

    export FOO=BAR
    /usr/bin/foo-bar

Вариант 3. Модификация директивы **Exec=** в ярлыке запуска приложения:

.. code-block:: bash

    env FOO=BAR /usr/bin/foo-bar

.. index:: git, vcs, configuration
.. _git-configuration:

Как правильно настроить Git для работы?
===========================================

Сначала укажем своё имя и адрес электронной почты:

.. code-block:: bash

    git config --global user.name "Your Name"
    git config --global user.email email@example.org

Установим :ref:`предпочитаемый текстовый редактор <editor-git>` для работы с коммитами:

.. code-block:: bash

    git config --global core.editor vim