### Валидация имени пропускает спецсимволы, цифры и пустые пробелы

**Симптом.** Пользователь может ввести в поле «Имя» значение `123`, `!@#$%`, (пробелы) или `<script>alert('xss')</script>`, и валидация это пропустит. В результате на бэкенд уйдёт некорректное имя, а в случае HTML-инъекции — возможна XSS-атака (поскольку данные вставляются через `innerHTML` без экранирования).

**Причина.** Функция `validateName` проверяет только длину после `trim()`, но не проверяет содержимое:

function validateName(name) {

  if (!name || name.trim().length === 0) {

    return 'Имя обязательно для заполнения';

  }

  if (name.trim().length < 2) {

    return 'Имя должно содержать минимум 2 символа';

  }

  return null; // ← пропускает всё остальное

}

**Исправление.** Добавить проверку на допустимые символы (буквы, пробел, дефис) и экранировать HTML при рендере:

function validateName(name) {

  if (!name || name.trim().length === 0) {

    return 'Имя обязательно для заполнения';

  }

  const trimmed = name.trim();

  if (trimmed.length < 2) {

    return 'Имя должно содержать минимум 2 символа';

  }

  if (trimmed.length > 100) {

    return 'Имя слишком длинное (максимум 100 символов)';

  }

  // Разрешаем буквы (включая кириллицу), пробелы, дефисы

  if (!/^[a-zA-Zа-яА-ЯёЁ\s\-]+$/.test(trimmed)) {

    return 'Имя может содержать только буквы, пробелы и дефисы';

  }

  return null;

}

// Функция экранирования HTML

function escapeHtml(text) {

  const div = document.createElement('div');

  div.textContent = text;

  return div.innerHTML;

}

// Использовать в renderSlotCard и renderModal:

function renderSlotCard(slot) {

  return `

    <div class="slot-card" data-slot-id="${escapeHtml([slot.id](http://slot.id))}">

      <div class="slot-card__header">

        <h3 class="slot-card__title">${escapeHtml(slot.title)}</h3>

        <span class="slot-card__date">${escapeHtml([slot.date](http://slot.date))}</span>

      </div>

      <div class="slot-card__time">🕐 ${escapeHtml(slot.time)}</div>

      <div class="slot-card__location">📍 ${escapeHtml(slot.location)}</div>

    </div>

  `;

}