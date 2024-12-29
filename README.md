const notification = document.querySelector('.notification');
const notificationText = document.querySelector('.notification__text');

const notificationAction = (opacity, transform) => {
  notification.style.opacity = opacity;
  notification.style.transform = `translateX(${transform}px)`;
};

const checkTextType = type => {
  switch (type) {
    case 'success':
      return '#00ff00';
    case 'error':
      return '#ff0000';
    default:
      return '#000';
  }
};

export const openNotification = (text, textType) => {
  notificationText.innerHTML = text;
  notificationText.style.color = checkTextType(textType);

  notificationAction(1, -350);
  setTimeout(() => {
    notificationAction(0, 0);
  }, 3000);
};
