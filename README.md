import { Image, Paragraph, Button, Input, Link } from '../components';
import { routeCatalog } from '../routes';
import { addProduct, removeProduct } from '../utils';

const Card = ({ className, mode, ...props } = {}) => {
  const isCatalog = mode === 'catalog';
  const isProduct = mode === 'product';
  const isBag = mode === 'bag';

  const card = document.createElement(isCatalog || isBag ? 'li' : 'div');
  const image = Image({
    className: isCatalog ? 'card__image' : 'product__image',
    src: props.image,
    title: props.name,
  });
  const details = document.createElement('div');
  const name = Paragraph({
    className: isCatalog ? 'card__name' : 'product__name',
    textContent: props.name,
  });
  const price = Paragraph({ textContent: props.price });

  card.className = className ? className : isCatalog ? 'card' : 'product';
  details.className = isCatalog ? 'card__details' : 'product__details';

  for (const key of Object.keys(props)) {
    card[key] = props[key];
  }

  const catalogProcess = () => {
    const range = Input({ type: 'range', min: 285, max: 410 });
    const openCard = () => history.push(`${routeCatalog}/${props.id}`);

    image.addEventListener('click', openCard);
    name.addEventListener('click', openCard);
    range.onchange = (e) => (card.style.width = `${e.target.value}px`);

    details.append(range);
  };

  const productProcess = () => {
    const count = Paragraph({
      textContent: props.count ? `Count: ${props.count}` : '',
    });
    const bagProcess = Button({
      textContent: isProduct ? 'Add to Bag' : 'Remove from Bag',
    });
    const label = document.createElement('label');
    const color = isProduct
      ? Input({ type: 'color' })
      : document.createElement('div');
    const link = Link({
      href: props.link,
      title: 'Similar product',
      textContent: isProduct ? `Similar product: ${props.link}` : '',
      target: '_blank',
      rel: 'prerender',
    });

    label.textContent = 'Color:';
    color.className = 'product__color';
    color.style.backgroundColor = props.color;

    bagProcess.addEventListener('click', () =>
      isProduct
        ? addProduct({
          id: props.id,
          image: props.image,
          name: props.name,
          price: props.price,
          color: color.value,
        })
        : removeProduct(props.id)
    );

    label.append(color);
    details.append(count, label, bagProcess, link);
  };

  details.append(name, price);
  card.append(image, details);

  isCatalog ? catalogProcess() : productProcess();

  return card;
};

export default Card;
