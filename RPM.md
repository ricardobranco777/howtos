
List packages listing vendor:

`rpm -qa --queryformat "%-32{VENDOR} %-48{NAME} %-32{VERSION} %{arch}\n" | sort`
