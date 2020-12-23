# Container & Presentational

Подход при котором компоненты разделяются на те, которые отвечают за логику и работу с данными - контейнеры, и на те, которые отвечают за представление. Таким образом можно отделать компоненты, составляющие дизайн-систему, от компонентов, реализующих бизнес-логику.

Например, компонент `GeoLocation` отвечает за отображение данных геолокации, а `GeoLocationContainer` за получение этих данных и передачу их `GeoLocation`:

```js
function GeoLocation({ latitude, longitude }) {
  return (
    <div>
      <h1>Geolocation:</h1>
      <div>Latitude: {latitude}</div>
      <div>Longitude: {longitude}</div>
    </div>
  );
}

class GeolocationContainer extends React.Component {
  state = {
    latitude: null,
    longitude: null
  };

  componentDidMount() {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(this.handleSuccess);
    }
  }

  handleSuccess = ({ coords: { latitude, longitude } }) => {
    this.setState({
      latitude,
      longitude
    });
  };

  render() {
    return <GeoLocation {...this.state} />;
  }
}
```

# Higher Order Components

Компоненты высшего порядка появились как попытка решить ту же проблему, что решали **миксины**, но, избежав проблем, которые **миксины** создавали, - разделать общую логику между компонентами. О компонентах высшего порядка можно думать, как, например, о частичном примении функции: они принимают компонент и возвращают его с частично переданными пропами; с функциональными компонентами это наиболее наглядно: **HoC** возвращает новую функцию с оставшимися аргументами. Другой способ думать о **HoC** - это как о декораторах функций: они возвращают компоненты с тем же интерфейсом, что и переданным, но с какой-то дополнительно логикой сверху.

```js
function Square({ width, color }) {
  return (
    <div style={{ width, height: width, backgroundColor: color }} />
  );
}

function withWindowWidth(Component) {
  return class {
    state.width = window.innerWidth;

    componentDidMount() {
      window.addEventListener('resize', this.handleResize);
    }
    
    componentWillUnmount() {
      window.removeEventListener('resize', this.handleResize);
    }

    handleResize = () => {
      this.setState({ width: window.innerWidth });
    }

    render() {
      return <Component {...this.props} {...this.state} />
    }
  };
}

const SquareWithWindowWidth = withWindowWidth(Square);
```

Также компоненты высшего порядка удобно использовать с контекстом в тех случаях, когда нужно держать компоненты изолированными от контекста, которым они пользуются. Например контекст отвечает за полуение данных из API, которые необходжимы какому-либо компоненту. Для этого компонента следует создать **HoC**, который будет отвечать за взаимодействие с контекстом и преобразование данных, полученных из него, в пропы, соответствующие интерфейсу компонента. Таким образом пользователи контекста будут защищены от изменений его интерфейса или логики работы в будущем.

# Function as a Child

Подход используется в тех случаях, когда родительский компонент написан так, что он не может знать, какие именно потомки будут ему переданы. То есть с помощью функции в качестве потомка можно получить полный контроль над рендерингом потомков какого-либо компонента снаружи.

Например, это может быть полезно для компонентов-контейнеров, которые отвечают только за работу с данными:

```js
class Fetch extends React.Component {
  state = {
    data: null
  };

  componentDidMount() {
    this.fetchData();
  }

  fetchData = async () => {
    const { url } = this.props;
    const response = await fetch(url);
    const { results } = await response.json();

    this.setState({ data: results[0] });
  };

  render() {
    const { children } = this.props;

    return children(this.state.data);
  }
}

export default function App() {
  function renderData(quiz) {
    if (!quiz) {
      return '...Fetching';
    }

    return (
      <h1>{ quiz.question }</h1>
    );
  }

  return (
    <Fetch url="https://opentdb.com/api.php?amount=1">
      { renderData }
    </Fetch>
  );
}
```
