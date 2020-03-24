<table>
  <thead>
    <tr>
    <th>English</th>
    <th>Russian</th>
    </tr>
  </thead>
  <tbody>
  {{ $url := "/content/notebook-phrases/voc.csv" }}
  {{ $sep := "," }}
  {{ range $i, $r := getCSV $sep $url }}
    <tr>
      <td>{{ index $r 0 }}</td>
      <td>{{ index $r 1 }}</td>
    </tr>
  {{ end }}
  </tbody>
</table>