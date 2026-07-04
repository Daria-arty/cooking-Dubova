### Баг 2. Прокрутка страницы за открытой модалкой (scroll bleed)

**Симптом.** На мобильном устройстве при открытой модалке пользователь может пальцем прокручивать страницу под затемнением — карточки расписания «уезжают» вверх-вниз. Это выглядит как баг, сбивает с толку и может привести к случайному закрытию модалки (если палец соскользёт на overlay).

**Причина.** При открытии модалки не блокируется прокрутка `body`. CSS `position: fixed` на overlay не предотвращает скролл фона на iOS/Android.

**Исправление.** Блокировать скролл при открытии и разблокировать при закрытии:

function openModal(slot) {

  if (isModalOpen) return;

  selectedSlot = slot;

  isModalOpen = true;

  renderModal(slot);

  [document.body.style](http://document.body.style).overflow = 'hidden'; // ← добавить

  [document.body.style](http://document.body.style).position = 'fixed';  // ← добавить (для iOS)

  [document.body.style](http://document.body.style).width = '100%';      // ← добавить

}

function closeModal() {

  if (!isModalOpen) return;

  

  if (autoCloseTimer) {

    clearTimeout(autoCloseTimer);

    autoCloseTimer = null;

  }

  

  modalRoot.innerHTML = '';

  isModalOpen = false;

  selectedSlot = null;

  

  [document.body.style](http://document.body.style).overflow = '';      // ← добавить

  [document.body.style](http://document.body.style).position = '';      // ← добавить

  [document.body.style](http://document.body.style).width = '';         // ← добавить

}

function handleSlotCardClick(event) {

  const card = [event.target](http://event.target).closest('.slot-card');

  if (!card) return;

  

  const slotId = card.dataset.slotId;

  const slot = currentSlots.find(s => [s.id](http://s.id) === slotId);

  

  if (!slot || isModalOpen) return;

  

  openModal(slot); // ← заменить прямой вызов renderModal

}