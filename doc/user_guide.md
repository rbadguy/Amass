
# [![OWASP Logo](https://github.com/OWASP/Amass/blob/master/images/owasp_logo.png) OWASP Amass](https://www.owasp.org/index.php/OWASP_Amass_Project)

![Network graph](https://github.com/OWASP/Amass/blob/master/images/network_06092018.png "Amass Network Mapping")

----

## Using the Tool Suite

The most basic use of the tool, which includes reverse DNS lookups and name alterations:
```
$ amass -d example.com
```

The example below is a good place to start with amass:
```
$ amass -src -ip -brute -min-for-recursive 3 -d example.com
[Google] www.example.com
[VirusTotal] ns.example.com
...
13139 names discovered - archive: 171, cert: 2671, scrape: 6290, brute: 991, dns: 250, alt: 2766
```

Add some additional domains to the enumeration:
```
$ amass -d example1.com,example2.com -d example3.com
```

You will need a config file to use your API keys with Amass. See the [Example Configuration File](https://github.com/OWASP/Amass/blob/master/examples/amass_config.ini) for more details.


Switches available through the amass CLI:

| Flag | Description | Example |
|------|-------------|---------|
| -active | Enable active recon methods | amass -active -d example.com net -p 80,443,8080 |
| -bl  | Blacklist undesired subdomains from the enumeration | amass -bl blah.example.com -d example.com |
| -blf | Identify blacklisted subdomains from a file | amass -blf data/blacklist.txt -d example.com |
| -brute | Perform brute force subdomain enumeration | amass -brute -d example.com |
| -config | Path to the INI configuration file | amass -config amass_settings.ini |
| -d   | Provide a domain name to include in the enumeration | amass -d example.com |
| -df  | Specify the domains to be enumerated via text file | amass -df domains.txt |
| -do  | Write all the data operations to a JSON file | amass -do data.json -d example.com |
| -ef  | Path to a file providing data sources to exclude | amass -ef exclude.txt -d example.com |
| -exclude | Data source names separated by commas to be excluded | amass -exclude crtsh -d example.com |
| -h   | Show the amass usage information | amass -h |
| -if  | Path to a file providing data sources to include | amass -if include.txt -d example.com |
| -include-unresolvable | Output DNS names that did not resolve | amass -include-unresolvable -d example.com |
| -include | Data source names separated by commas to be included | amass -include crtsh -d example.com |
| -ip  | Print IP addresses with the discovered names | amass -ip -d example.com |
| -json | All discoveries written as individual JSON objects | amass -json out.json -d example.com |
| -list | Print the names of all available data sources | amass -l |
| -log | Log all error messages to a file | amass -log amass.log -d example.com |
| -min-for-recursive | Number of subdomain names required for recursive brute forcing to begin | amass -brute -min-for-recursive 3 -d example.com |
| -noalts | Disable alterations of discovered names | amass -noalts -d example.com |
| -passive | A purely passive mode of execution | amass --passive -d example.com |
| -norecursive | Disable recursive brute forcing | amass -brute -norecursive -d example.com |
| -o   | Write the results to a text file | amass -o out.txt -d example.com |
| -oA  | Output to all available file formats with prefix | amass -oA amass_scan -d example.com |
| -r   | Specify your own DNS resolvers | amass -r 8.8.8.8,1.1.1.1 -d example.com |
| -rf  | Specify DNS resolvers with a file | amass -rf data/resolvers.txt -d example.com |
| -src | Print data sources for the discovered names | amass -src -d example.com |
| -T   | Timing templates 0 (slowest) through 5 (fastest) (default 3) | amass -T 5 -d example.com |
| -version | Print the version number of amass | amass -version |
| -w   | Change the wordlist used during brute forcing | amass -brute -w wordlist.txt -d example.com |

#### amass.netdomains

**Caution:** If you use the amass.netdomains tool, it will attempt to reach out to every IP address within the identified infrastructure and obtain domains names from reverse DNS requests and TLS certificates. This is "loud" and can reveal your reconnaissance activities to the organization being investigated.

| Flag | Description | Example |
|------|-------------|---------|
| -org | Search string provided against AS description information | amass.netdomains -org Facebook |
| -asn  | ASNs separated by commas (can be used multiple times) | amass.netdomains -asn 13374,14618 |
| -cidr | CIDRs separated by commas (can be used multiple times) | amass.netdomains -cidr 104.154.0.0/15 |
| -addr | IPs and ranges (192.168.1.1-254) separated by commas | amass.netdomains -addr 192.168.2.1-64 |
| -p | Ports separated by commas (default: 443) | amass.netdomains -cidr 104.154.0.0/15 -p 443,8080 |
| -whois | All discovered domains are run through reverse whois | amass.netdomains -whois -asn 13374 |

#### amass.viz

Create enlightening network graph visualizations that provide structure to the information you gather. This tool requires an input file generated by the amass '-do' flag.

Switches for outputting the DNS and infrastructure findings as a network graph:

| Flag | Description | Example |
|------|-------------|---------|
| -maltego | Output a Maltego Graph Table CSV file | amass.viz -maltego net.csv -i data_ops.json |
| -d3  | Output a D3.js v4 force simulation HTML file | amass.viz -d3 net.html -i data_ops.json |
| -gexf | Output to Graph Exchange XML Format (GEXF) | amass.viz -gephi net.gexf -i data_ops.json |
| -graphistry | Output Graphistry JSON | amass.viz -graphistry net.json -i data_ops.json |
| -visjs | Output HTML that employs VisJS | amass.viz -visjs net.html -i data_ops.json |


#### amass.db

Have amass send all the DNS and infrastructure information gathered to a graph database. This tool requires an input file generated by the amass '-do' flag.

```
$ amass.db -neo4j neo4j:DoNotUseThisPassword@localhost:7687 -i data_ops.json
```

## Integrating OWASP Amass into Your Work

If you are using the amass package within your own Go code, be sure to properly seed the default pseudo-random number generator:
```go
import(
    "fmt"
    "math/rand"
    "time"

    "github.com/OWASP/Amass/amass"
)

func main() {
    // Seed the default pseudo-random number generator
	rand.Seed(time.Now().UTC().UnixNano())

	enum := amass.NewEnumeration()
	// Setup the most basic amass configuration
	enum.Config.AddDomain("example.com")
	
	go func() {
		for result := range enum.Output {
			fmt.Println(result.Name)
		}
	}()
	
	enum.Start()
}
```

## Importing OWASP Amass Results into Maltego

1. Output your Amass enumeration data using the '-do' flag:
```
$ amass -src -ip --active -brute -do owasp.json -d owasp.org
```

2. Convert the Amass data into a Maltego graph table CSV file:
```
$ amass.viz -i owasp.json --maltego owasp.csv
```

3. Import the CSV file with the correct Connectivity Table settings:

![Connectivity table](https://github.com/OWASP/Amass/blob/master/images/maltego_graph_import_wizard.png "Connectivity Table Settings")

4. All the Amass findings will be brought into your Maltego Graph:

![Maltego results](https://github.com/OWASP/Amass/blob/master/images/maltego_results.png "Maltego Results")


## Amass Terminal Capture 

Presented at Facebook (and shared publically) for the Sept. 2018 OWASP London Chapter meeting:

[![asciicast](https://asciinema.org/a/v6B1qdMRlLRUflpkwRPhvCTaY.png)](https://asciinema.org/a/v6B1qdMRlLRUflpkwRPhvCTaY)

