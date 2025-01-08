# google-serp-highlighter

![image](https://github.com/user-attachments/assets/b72621f0-5b96-4b80-ba4d-994d1aa62176)

![image](https://github.com/user-attachments/assets/1e6de065-0c2f-4da6-a992-748121ce3e15)

![image](https://github.com/user-attachments/assets/d485197c-ade3-4b47-812d-a41fb014525e)

![image](https://github.com/user-attachments/assets/8afe9071-7676-4514-a2f7-da1a454b62e3)


EN:
Java Script for highlighting personal sites and labeling ads in Google search results. Improves the visibility of personal sites (green highlighting) and ads (red circling) in desktop and mobile SERPs. Supports multiple languages and is easily customizable.

Highlighting your own domains and advertisements in Google search results.
Features:
— Your websites are highlighted in green;
— Advertisements are outlined with a red border (at least in RU & EN search results, easy to adapt for other languages - just add the necessary variant to the condition - node.textContent.toLowerCase().includes('sponsored') ||node.textContent.includes('Реклама');
— Works in both desktop and mobile search results.
It's convenient to work with it through the extension https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld
To make the code work through the User JavaScript and CSS extension, you need to allow "Run at page start" (the rightmost icon at the bottom of the JS code field)

URL pattern for application (input string above the JS code field):
https://www.google.com/,https://www.google.ru/,https://www.google.ae/*


RU:
Подсвечивание своих доменов и подсвечивание рекламы в выдаче Google.

Особенности:
— Свои сайты подсвечиваются зеленым;
— Реклама обводится красной рамкой (как минимум в RU & EN выдаче, доработать под другие языки легко - добавить в условие нужный вариант  - node.textContent.toLowerCase().includes('sponsored') ||node.textContent.includes('Реклама');
— Работает в ПК и Моб выдаче.

Удобно работать с ним через расширение https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld
Чтобы код заработал через расширение User JavaScript and CSS надо разрешить "Запуск на старте" (внизу поля с JS кодом крайний правый значек)
![image](https://github.com/user-attachments/assets/5800a718-3042-4ce6-8949-036330ee21e6)



URL-паттерн для применения (строка ввода над полем с JS-кодом) : 
https://www.google.com/*,https://www.google.ru/*,https://www.google.ae/*


****************************************************************************

The code itself: / Сам код:

function initializeAdHighlighter() {
  const myDomains = [
    'apple.com',
    'technodom.kz'
  ];

  function escapeRegExp(string) {
    return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  }

  const domainRegex = new RegExp(
    myDomains.map((domain) => `(^|\\.)${escapeRegExp(domain)}$`).join('|')
  );

  function highlightAd(element) {
    element.style.color = 'black';
    element.style.border = '2px solid red';
  }

  function checkAndHighlightDomain(link, topDiv) {
    let href = link.getAttribute('href');
    if (href) {
      try {
        let url = new URL(href);
        if (domainRegex.test(url.hostname)) {
          topDiv.style.backgroundColor = '#009159';
          topDiv.style.color = 'white';
        }
      } catch (e) {
        // Игнорируем некорректные URL
      }
    }
  }

  function markAds(node) {
    if (node.nodeType !== Node.ELEMENT_NODE) return;

    // Проверка на рекламу
    if (node.tagName === 'SPAN' || node.tagName === 'A') {
      let parentDiv = node.parentElement;
      if (node.tagName === 'SPAN') {
        if (
          node.textContent.toLowerCase().includes('sponsored') ||
          node.textContent.includes('Реклама')
        ) {
          highlightAd(parentDiv);
        }
      } else if (node.tagName === 'A') {
        let dataRw = node.getAttribute('data-rw');
        let dataPcu = node.getAttribute('data-pcu');
        if (
          (dataRw && dataRw.includes('aclk?sa=')) ||
          (dataPcu && dataPcu.includes('https://ad.doubleclick.net/'))
        ) {
          highlightAd(parentDiv);
        }
      }
    }

    // Проверка на соответствие доменам из списка
    if (node.tagName === 'A') {
      let topDiv = node.closest('div > div');
      if (topDiv) {
        checkAndHighlightDomain(node, topDiv);
      }
    }

    // Проверка на наличие ссылок с googleadservices.com
    if (
      node.tagName === 'A' &&
      node.href &&
      node.href.includes('googleadservices.com')
    ) {
      let adDiv = node.closest('div');
      if (adDiv) {
        highlightAd(adDiv);
      }
    }

    // Рекурсивно проверяем дочерние элементы
    node.childNodes.forEach(markAds);
  }

  const observer = new MutationObserver((mutations) => {
    mutations.forEach((mutation) => {
      if (mutation.type === 'childList') {
        mutation.addedNodes.forEach(markAds);
      } else if (mutation.type === 'attributes') {
        markAds(mutation.target);
      }
    });
  });

  const config = { childList: true, subtree: true, attributes: true };

  observer.observe(document.body, config);

  // Начальная маркировка существующих элементов
  markAds(document.body);

  // Функция для остановки наблюдения (если потребуется)
  function stopObserving() {
    observer.disconnect();
  }

  // Возвращаем функцию stopObserving, если нужно будет остановить наблюдение извне
  return stopObserving;
}

// Запускаем функцию после загрузки DOM
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', initializeAdHighlighter);
} else {
  initializeAdHighlighter();
}
