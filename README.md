class foreman {

    file {“foreman.list”:

        ensure => file,

        path => “/etc/apt/sources.list.d/foreman.list”,

        source => “puppet:///modules/foreman/foreman.list”,

        require => Exec[“update-init”],

}

    package {‘ruby1.9.1-dev’:

        ensure => installed,

        require => Exec[‘foreman.list’],

    }

    package {‘mysqlclient-dev’:

        ensure => installed,

        require => Exec[‘ruby1.9.1-dev’],

    }

    package {‘foreman’:

        ensure => installed,

        require => Exec[‘mysqlclient-dev’],

    }

    package {‘puppetmaster’:

        ensure => installed,

        require => Exec[‘foreman’],

    }

    package {‘puppetmaster-common’:

        ensure => installed,

        require => Exec[‘puppetmaster’],

    }

    file {‘foremanweb’:

        ensure => file,

        path => ‘/etc/apache2/sites-enable/puppetmaster’,

        source => “puppet:///modules/foreman/apache.conf”,

        require => Package[‘puppetmaster-common’],

    }

    file {“foremangem”:

        ensure => file,

        path => ‘/usr/share/foreman/Gemfile’,

        source => “puppet:///modules/foreman/apache.conf”,

        require => File[‘foremanweb’],

}

    file {“foremandb”:

        ensure => file,

        path => ‘/usr/share/foreman/config/database.yml’,

        source => “puppet:///modules/foreman/apache.conf”,

        require => File[‘foremangem’],

    }

    exec {“dbprod”:

        path => $::path,

        command => “mysql -u root --password=asd -e ´create database foremanprod;’”,

        require => File[‘foremandb’],

    }

    exec {“dbdev”:

        path => $::path,

        command => “mysql -u root --password=asd -e ´create database foremandev;’”,

        require => Exec[‘dbprod’],

    }


    exec {“dbtest”:

        path => $::path,

        command => “mysql -u root --password=asd -e ´create database foremantest;’”,

        require => Exec[‘dbdev’],

    }

    exec {‘foremanbundlegem’:

        path => $::path,

        cwd => “/usr/share/foreman”,

        command => “gem install json && bundle”,

        require => Exec[‘dbtest’],

    }

    exec {‘foremandbcreate’:

        path => $::path,

        cwd => “/usr/share/foreman”,

        command => “RAILS_ENV=production bundle exec rake db:migrate”,

        require => Exec[‘foremanboundlegem’],

    }
}
