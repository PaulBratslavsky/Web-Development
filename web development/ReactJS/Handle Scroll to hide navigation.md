# Scroll event in react
```javascript


const [visible, setVisible] = useState(false);
  
const page = useRef(null);

  
useEffect(() => {

	const handleScroll = () => {
	const { current } = page;
	
	if (current.scrollTop === 0) setVisible(false);
	else setVisible(true);
}


const { current } = page;
current.addEventListener('scroll', handleScroll)

return () => current.removeEventListener('scroll', handleScroll)

}, [])
```
