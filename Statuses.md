| Symbol | Meaning                                        |
| ------ | ---------------------------------------------- |
| 🌱     | Seedlings for very rough and early ideas       |
| 🌿     | Budding for work I've cleaned up and clarified |
| 🌳     | Evergreen for work that is reasonably complete |

_[Maggie Appleton's Post on Note Taking](https://maggieappleton.com/garden-history) & [Further Info](https://github.com/MaggieAppleton/digital-gardeners#theory-philosophy-and-navel-gazing)_


```dataview
TABLE WITHOUT ID file.link as Name, gardening as Garden, join(tags) as Tags
WHERE gardening = "🌱"
SORT file.name ASC
```

```dataview
TABLE WITHOUT ID file.link as Name, gardening as Garden, join(tags) as Tags
WHERE gardening = "🌿"
SORT file.name ASC
```

```dataview
TABLE WITHOUT ID file.link as Name, gardening as Garden, join(tags) as Tags
WHERE gardening = "🌳"
SORT file.name ASC
```


```dataview
TABLE WITHOUT ID file.link as Name, gardening as Garden, join(tags) as Tags
WHERE gardening = "🌱" OR gardening = "🌿" OR gardening = "🌳"
SORT file.name ASC
```