# google-serp-highlighter

# RU
Подсвечивание своих доменов и подсвечивание рекламы в выдаче Google.

Особенности:

— Свои сайты подсвечиваются зеленым;

— Реклама обводится красной рамкой (как минимум в RU & EN выдаче, доработать под другие языки легко - добавить в условие нужный вариант  - node.textContent.toLowerCase().includes('sponsored') ||node.textContent.includes('Реклама');

— Работает в ПК и Моб выдаче.

Удобно работать с ним через расширение https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld
Чтобы код заработал через расширение User JavaScript and CSS надо разрешить "Запуск на старте" (внизу поля с JS кодом крайний правый значек)
![image](https://github.com/user-attachments/assets/5800a718-3042-4ce6-8949-036330ee21e6)



URL-паттерн для применения (строка ввода над полем с JS-кодом) : 
```https://www.google.com/*,https://www.google.ru/*,https://www.google.ae/*```




## Скриншоты

![image](https://github.com/user-attachments/assets/b72621f0-5b96-4b80-ba4d-994d1aa62176) ![image](https://github.com/user-attachments/assets/1e6de065-0c2f-4da6-a992-748121ce3e15) ![image](https://github.com/user-attachments/assets/d485197c-ade3-4b47-812d-a41fb014525e) ![image](https://github.com/user-attachments/assets/8afe9071-7676-4514-a2f7-da1a454b62e3)


## Сам код:

```
function initializeAdHighlighter() {
  const myDomains = [
    'apple.com',
    'technodom.kz'
  ];
const domainRegex = new RegExp(`(^|\\.)(?:${myDomains.map(d => d.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).join('|')})$`);

  // Кэшируем селекторы и проверки
  const isSponsored = (text) => text.toLowerCase().includes('sponsored') || text.includes('Реклама');
  const isAdLink = (node) => {
    const dataRw = node.getAttribute('data-rw');
    const dataPcu = node.getAttribute('data-pcu');
    return (dataRw?.includes('aclk?sa=') || dataPcu?.includes('https://ad.doubleclick.net/'));
  };

  // Объединяем стили в один объект
  const styles = {
    ad: { color: 'black', border: '2px solid red' },
    domain: { backgroundColor: '#009159', color: 'white' }
  };

  // Применение стилей одним вызовом
  const applyStyles = (element, styleObj) => Object.assign(element.style, styleObj);

  function checkAndHighlightDomain(link, topDiv) {
    try {
      const url = new URL(link.href);
      if (domainRegex.test(url.hostname)) applyStyles(topDiv, styles.domain);
    } catch (e) {}
  }

  function markAds(node, isRoot = false) {
    if (node.nodeType !== 1) return; // Только элементы

    // Обрабатываем только конкретные теги
    if (node.tagName === 'SPAN' || node.tagName === 'A') {
      const parentDiv = node.parentElement;
      
      if (node.tagName === 'SPAN' && isSponsored(node.textContent)) {
        applyStyles(parentDiv, styles.ad);
      } else if (node.tagName === 'A') {
        if (isAdLink(node) || node.href.includes('googleadservices.com')) {
          const adDiv = node.closest('div');
          if (adDiv) applyStyles(adDiv, styles.ad);
        }
        // Проверка доменов только для ссылок
        const topDiv = node.closest('div > div');
        if (topDiv) checkAndHighlightDomain(node, topDiv);
      }
    }

    // Ограничиваем рекурсию для крупных узлов
    if (isRoot || node.childElementCount < 50) {
      for (const child of node.children) markAds(child);
    }
  }

  // Оптимизированный MutationObserver
  const observer = new MutationObserver((mutations) => {
    const visited = new Set(); // Избегаем повторной обработки
    for (const mutation of mutations) {
      if (mutation.type === 'childList') {
        for (const node of mutation.addedNodes) {
          if (node.nodeType === 1 && !visited.has(node)) {
            visited.add(node);
            markAds(node);
          }
        }
      } else if (mutation.type === 'attributes' && !visited.has(mutation.target)) {
        visited.add(mutation.target);
        markAds(mutation.target);
      }
    }
  });

  // Запускаем с оптимизированным конфигом
  observer.observe(document.body, { childList: true, subtree: true, attributes: true, attributeFilter: ['href', 'data-rw', 'data-pcu'] });

  // Начальная обработка с таймаутом для асинхронности
  setTimeout(() => markAds(document.body, true), 0);

  return () => observer.disconnect();
}

// Запуск с оптимизацией
document.readyState === 'loading' 
  ? document.addEventListener('DOMContentLoaded', initializeAdHighlighter)
  : setTimeout(initializeAdHighlighter, 0);

```

Автор: @sc00d
Телеграм-канал https://t.me/seregaseo


# EN
Java Script for highlighting personal sites and labeling ads in Google search results. Improves the visibility of personal sites (green highlighting) and ads (red circling) in desktop and mobile SERPs. Supports multiple languages and is easily customizable.

Highlighting your own domains and advertisements in Google search results.
Features:

— Your websites are highlighted in green;

— Advertisements are outlined with a red border (at least in RU & EN search results, easy to adapt for other languages - just add the necessary variant to the condition - node.textContent.toLowerCase().includes('sponsored') ||node.textContent.includes('Реклама');

— Works in both desktop and mobile search results.

It's convenient to work with it through the extension https://chromewebstore.google.com/detail/user-javascript-and-css/nbhcbdghjpllgmfilhnhkllmkecfmpld
To make the code work through the User JavaScript and CSS extension, you need to allow "Run at page start" (the rightmost icon at the bottom of the JS code field)

URL pattern for application (input string above the JS code field):
```https://www.google.com/,https://www.google.ru/,https://www.google.ae/*```

## Screenshot

![image](https://github.com/user-attachments/assets/b72621f0-5b96-4b80-ba4d-994d1aa62176) ![image](https://github.com/user-attachments/assets/1e6de065-0c2f-4da6-a992-748121ce3e15) ![image](https://github.com/user-attachments/assets/d485197c-ade3-4b47-812d-a41fb014525e) ![image](https://github.com/user-attachments/assets/8afe9071-7676-4514-a2f7-da1a454b62e3)

## The code itself

```
function initializeAdHighlighter() {
  const myDomains = [
    'apple.com',
    'technodom.kz'
  ];

  const domainRegex = new RegExp(`(^|\\.)(?:${myDomains.map(d => d.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).join('|')})$`);

  // Кэшируем селекторы и проверки
  const isSponsored = (text) => text.toLowerCase().includes('sponsored') || text.includes('Реклама');
  const isAdLink = (node) => {
    const dataRw = node.getAttribute('data-rw');
    const dataPcu = node.getAttribute('data-pcu');
    return (dataRw?.includes('aclk?sa=') || dataPcu?.includes('https://ad.doubleclick.net/'));
  };

  // Объединяем стили в один объект
  const styles = {
    ad: { color: 'black', border: '2px solid red' },
    domain: { backgroundColor: '#009159', color: 'white' }
  };

  // Применение стилей одним вызовом
  const applyStyles = (element, styleObj) => Object.assign(element.style, styleObj);

  function checkAndHighlightDomain(link, topDiv) {
    try {
      const url = new URL(link.href);
      if (domainRegex.test(url.hostname)) applyStyles(topDiv, styles.domain);
    } catch (e) {}
  }

  function markAds(node, isRoot = false) {
    if (node.nodeType !== 1) return; // Только элементы

    // Обрабатываем только конкретные теги
    if (node.tagName === 'SPAN' || node.tagName === 'A') {
      const parentDiv = node.parentElement;
      
      if (node.tagName === 'SPAN' && isSponsored(node.textContent)) {
        applyStyles(parentDiv, styles.ad);
      } else if (node.tagName === 'A') {
        if (isAdLink(node) || node.href.includes('googleadservices.com')) {
          const adDiv = node.closest('div');
          if (adDiv) applyStyles(adDiv, styles.ad);
        }
        // Проверка доменов только для ссылок
        const topDiv = node.closest('div > div');
        if (topDiv) checkAndHighlightDomain(node, topDiv);
      }
    }

    // Ограничиваем рекурсию для крупных узлов
    if (isRoot || node.childElementCount < 50) {
      for (const child of node.children) markAds(child);
    }
  }

  // Оптимизированный MutationObserver
  const observer = new MutationObserver((mutations) => {
    const visited = new Set(); // Избегаем повторной обработки
    for (const mutation of mutations) {
      if (mutation.type === 'childList') {
        for (const node of mutation.addedNodes) {
          if (node.nodeType === 1 && !visited.has(node)) {
            visited.add(node);
            markAds(node);
          }
        }
      } else if (mutation.type === 'attributes' && !visited.has(mutation.target)) {
        visited.add(mutation.target);
        markAds(mutation.target);
      }
    }
  });

  // Запускаем с оптимизированным конфигом
  observer.observe(document.body, { childList: true, subtree: true, attributes: true, attributeFilter: ['href', 'data-rw', 'data-pcu'] });

  // Начальная обработка с таймаутом для асинхронности
  setTimeout(() => markAds(document.body, true), 0);

  return () => observer.disconnect();
}

// Запуск с оптимизацией
document.readyState === 'loading' 
  ? document.addEventListener('DOMContentLoaded', initializeAdHighlighter)
  : setTimeout(initializeAdHighlighter, 0);

```

Author: Telegram @sc00d 
Telegram channel https://t.me/seregaseo



