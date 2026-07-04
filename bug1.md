### Баг 1. setTimeout не очищается при закрытии модалки — новая модалка закрывается сама

**Симптом.** Пользователь успешно записывается на мастер-класс, видит зелёное сообщение «Вы успешно записаны…», но решает закрыть модалку вручную (крестиком или кликом по overlay) до истечения 2 секунд. Затем он открывает **другую** модалку для записи на иной слот — и она **автоматически закрывается** через оставшееся время (например, через 0.5–1 секунду), не давая заполнить форму.

**Причина.** В `handleFormSubmit` создаётся `setTimeout` для автозакрытия, но его ID нигде не сохраняется. При ручном закрытии `closeModal()` не отменяет этот таймер. Когда пользователь открывает новую модалку, старый таймер срабатывает и вызывает `closeModal()` повторно.

**Исправление.** Сохранять ID таймера в переменной состояния и очищать его в `closeModal`:

// State

let currentSlots = [];

let isModalOpen = false;

let selectedSlot = null;

let autoCloseTimer = null; // ← добавить

function closeModal() {

  if (!isModalOpen) return;

  

  if (autoCloseTimer) {          // ← добавить

    clearTimeout(autoCloseTimer);

    autoCloseTimer = null;

  }

  

  modalRoot.innerHTML = '';

  isModalOpen = false;

  selectedSlot = null;

}

// В handleFormSubmit, ветка успеха:

showMessage(

  `Вы успешно записаны на мастер-класс! Ваш номер брони: ${result.data.bookingId}`,

  'success'

);

autoCloseTimer = setTimeout(() => {   // ← сохранить ID

  closeModal();

}, 2000);